# SDK testing (frontend surface)

Verification criteria, fixture design, and definition of done for the **Python SDK** surface —
the `<Lang>Analysis` facade and its `<Lang>Codeanalyzer` wrapper. The **analyzer-side** gates
(symbol-table, call-graph, caching tests run against the binary) live in the companion
**codeanalyzer-backend** skill's `references/testing-and-validation.md`. Keep the two in sync
but scoped to their own surface.

> **Never fake verification.** Every test file below must actually run under `pytest`. Mocked
> tests always run (no binary needed); E2E tests skip cleanly when the binary is absent but
> must pass when it is present. Don't claim a suite passed without running it.

---

## 1. Fixture design (SDK-side)

Fixture location: `tests/resources/<lang>/`.

- A small `analysis.json` (or a directory containing one) that can be loaded without running
  the binary, under `tests/resources/<lang>/analysis_json/`. Used by mocked tests.
- A real project fixture under `tests/resources/<lang>/application/` for E2E tests. This can
  be the same fixture the analyzer's own `go test` (etc.) uses if the SDK lives next to the
  analyzer repo; otherwise replicate a subset. It must satisfy the analyzer-side fixture
  minimums (multi-file unit, exported + unexported symbols, a named call-graph edge,
  language-specific fields) — see the backend skill's `testing-and-validation.md § 1`.

---

## 2. Mocked tests

Location: `tests/analysis/<lang>/test_<lang>_analysis.py`

**Pattern:** monkeypatch the wrapper's `_run_and_parse()` (or equivalent) to return a
pre-built `<Lang>Application` from a fixture JSON. Tests never invoke the binary.

```python
@pytest.fixture
def fake_app():
    return <Lang>Application(**json.loads(FIXTURE_JSON))

@pytest.fixture
def analysis(tmp_path, monkeypatch, fake_app):
    monkeypatch.setattr(
        "<Lang>Codeanalyzer",
        "_run_and_parse",
        lambda *a, **kw: fake_app,
    )
    return <Lang>Analysis(project_dir=tmp_path, ...)

def test_get_symbol_table_non_empty(analysis, fake_app):
    assert analysis.get_symbol_table() == fake_app.symbol_table
```

**Minimum mocked test coverage:**

- `source_code` mode raises `CldkInitializationException` (if not supported for this
  language).
- `get_symbol_table()` returns the expected dict.
- `get_file("<known_path>")` returns the correct `<Lang>File`.
- `get_all_types()` / `get_all_callables()` iterate across files correctly.
- `get_call_graph()` returns a `nx.DiGraph` with the expected node and edge counts.
- `get_callers(sig)` / `get_callees(sig)` return correct subsets.
- Pydantic round-trip: `<Lang>Application(**json.loads(app.model_dump_json()))` succeeds.
- Language-specific alias properties (e.g. `GoFile.types`, `GoFile.package_name`) return
  expected values.
- Null JSON fields coerce correctly — pass a fixture JSON with `null` list/dict fields and
  confirm the model loads without error (tests the `_NullSafeBase` guard — see
  `python-sdk-wiring.md § Common pitfalls`).

---

## 3. E2E tests

Location: `tests/analysis/<lang>/test_<lang>_e2e.py` (separate file from mocked tests)

**Why E2E tests alongside mocked tests:** mocked tests confirm the facade logic is correct
*given* the right `analysis.json`. They cannot catch CLI-flag bugs (e.g., the wrapper
sending `--analysis-level symbol_table` when the binary expects `--analysis-level 1`), null
serialization mismatches, or real schema fields missing from the output. E2E tests catch
that entire class.

**Skip pattern** — tests must be skippable when the binary is absent (CI without the
language toolchain must not fail):

```python
pytestmark = pytest.mark.skipif(
    shutil.which("codeanalyzer-<lang>") is None,
    reason="codeanalyzer-<lang> not found on PATH",
)
```

**Helper pattern** — one helper that constructs a real `<Lang>Analysis` pointing at the
fixture and a `tmp_path` output dir:

```python
def _analysis(tmp_path, level=AnalysisLevel.symbol_table):
    return <Lang>Analysis(
        project_dir=FIXTURE_DIR,
        analysis_backend_path=None,
        analysis_json_path=tmp_path,   # isolates each test's output
        analysis_level=level,
        eager_analysis=True,           # always re-run; don't depend on cached state
    )
```

**Minimum E2E test coverage:**

- All expected source files appear as keys in `symbol_table`.
- No key is an absolute path or starts with `..`.
- `<Lang>Application(**json.load(open(tmp_path / "analysis.json")))` succeeds (Pydantic
  round-trip on the real output file).
- Every language-specific schema field exercised by the fixture has an E2E assertion with a
  specific expected value — not just "non-empty".
- Named expected call-graph edge is present (level-2 run).
- Cross-package edge is present (level-2 run).
- No dangling call-graph nodes.
- Cache idempotency (SDK-level skip): first run with `eager=True` seeds the cache; second run
  with `eager=False` does not rewrite `analysis.json` (assert `st_mtime` unchanged after
  `time.sleep(0.05)`). This exercises the facade's `_check_existing_analysis()` skip, the
  third caching layer described in the backend skill's `SKILL.md`.

---

## 4. Definition of done (SDK surface)

- [ ] Mocked SDK tests pass under `pytest` (backend patched).
- [ ] `CLDK(language="<lang>").analysis(project_path=<fixture>).get_symbol_table()` is
  non-empty when run with the binary on PATH.
- [ ] `get_call_graph()` returns a NetworkX DiGraph with no dangling nodes.
- [ ] E2E tests exist and skip cleanly when the binary is absent.
- [ ] `pyproject.toml [tool.backend-versions]` is pinned to the released version.
- [ ] All changes are on the `add-<lang>-support` branch; the tree is clean.
