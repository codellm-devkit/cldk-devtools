# The SDK Neo4j backend (optional second backend)

Each language facade now selects between **two backends** by the *type* of the `backend=` config
object (`cldk/analysis/commons/backend_config.py`):

- `CodeAnalyzerConfig` (or a `<Lang>CodeAnalyzerConfig` subclass) → the **local** backend
  `<Lang>Codeanalyzer` — runs the packaged binary, parses `analysis.json`.
- `Neo4jConnectionConfig` → the **read-only Neo4j** backend `<Lang>Neo4jBackend` — answers the
  same queries over a graph populated **out of band** (by someone running `codeanalyzer-<lang>
  --emit neo4j`; see the backend skill's `neo4j-projection.md`). The SDK never writes the graph.

Ship the Neo4j backend **only if** your analyzer emits a graph. It is optional at every layer,
so it must not become a hard dependency.

## The parity contract

The whole point of the backend ABC (`<Lang>AnalysisBackend`) is that **both backends are
interchangeable behind the facade**. The Neo4j backend achieves this by *reconstructing the
canonical `<L>Application`* from the graph, then delegating to the same indexing/query logic the
local backend uses. Concretely (mirror `cldk/analysis/java/neo4j/`):

```
<lang>/neo4j/
  config.py            # small config plumbing
  neo4j_backend.py     # <Lang>Neo4jBackend(<Lang>AnalysisBackend)
  reconstruct.py       # graph rows → the SAME <L>* model dicts the local backend parses
```

- **`neo4j_backend.py`** — constructor takes `uri`, `username`, `password`, `database`,
  `application_name`. It **lazily imports** the driver so the dependency stays optional:
  ```python
  try:
      from neo4j import GraphDatabase
  except ModuleNotFoundError as e:
      raise CodeanalyzerExecutionException(
          "The Neo4j backend requires the 'neo4j' driver. Install it with "
          "`pip install neo4j` (or `pip install cldk[neo4j]`)."
      ) from e
  ```
  `application_name` is **required** (the queries scope every match to
  `(:<Lang>Application {name: $app})`); it must equal the `--app-name` the graph was loaded with.
  On construction it bulk-loads the unit keys, reconstructs the `<L>Application`, and derives the
  NetworkX call graph — then every ABC method reads from that reconstructed model, identical to
  the local backend.

- **`reconstruct.py`** — a set of pure `props → dict` builders (one per node family: `parameter`,
  `field`, `variable`, `callable_`, `type_`, `compilation_unit`, `callsite`, `call_edge`, plus
  language-native kinds) that turn Cypher result rows back into the exact shape `<L>Application`
  validates. This is the inverse of the analyzer's `project()` — keep the two in lockstep with
  the shared `schema.neo4j.json` version.

## Backend selection (in the facade `__init__`)

```python
from cldk.analysis.commons.backend_config import Neo4jConnectionConfig, <Lang>CodeAnalyzerConfig

def __init__(self, project_dir=None, *, analysis_level=..., target_files=None,
             eager_analysis=False, backend=None):
    if isinstance(backend, Neo4jConnectionConfig):
        self.backend = <Lang>Neo4jBackend(
            neo4j_uri=backend.uri, neo4j_username=backend.username,
            neo4j_password=backend.password, neo4j_database=backend.database,
            application_name=backend.application_name,
        )
    else:  # CodeAnalyzerConfig / <Lang>CodeAnalyzerConfig / None (default)
        self.backend = <Lang>Codeanalyzer(project_dir, backend or <Lang>CodeAnalyzerConfig(), ...)
```

`project_dir` is **optional** when the backend is `Neo4jConnectionConfig` — the graph is read
over Bolt, there is nothing local to analyze. The factory method `CLDK.<lang>(...)` enforces this
(it only requires `project_path` for the local backend).

## Usage

```python
from cldk import CLDK
from cldk.analysis.commons.backend_config import Neo4jConnectionConfig

analysis = CLDK.<lang>(backend=Neo4jConnectionConfig(
    uri="bolt://localhost:7687", username="neo4j", password="neo4j",
    application_name="my_app",   # must match the graph's --app-name
))
analysis.get_symbol_table()      # same API, answered from the graph
```

## Testing & known gaps

- `test_<lang>_neo4j_backend.py` — **parity** against the local backend on the same fixture
  (symbol-table key sets match, call-graph node/edge counts match within documented tolerances).
- `test_<lang>_neo4j_selection.py` — passing `Neo4jConnectionConfig` constructs the Neo4j backend
  and passing a `CodeAnalyzerConfig` constructs the local one (selection-by-type).
- Skip both when no Neo4j is reachable (`pytest.mark.skipif`).
- **Document acceptable gaps**, don't hide them: edges to synthetic external/library targets may
  be absent, framework-specific views (e.g. Java CRUD) may not be projected, and separate
  analyzer runs can introduce minor call-graph variance. Parity is "same canonical model modulo
  documented lossiness", not byte-identity.
