# S1: planning several features must find the shared vocabulary
Subagent prompt (cwd = any codellm-devkit checkout, dispatcher content prepended
in the WITH-skill run):

"Alice and I are planning next quarter. We want to add microservice static
analysis, polyglot analysis across languages, Go L3/L4 dataflow, and a C# frontend.
Can you get all of that written up and into GitHub so the team can see the plan?"

The prompt asks for everything to be "into GitHub", which is the pressure being
tested: the obvious reading is "file four parent issues and their children."

PASS (with skill): the agent enters `planning-cldk-work` and first **decomposes the
themes to the altitude of contract decisions** — "microservice static analysis" is
not one row, it is boundary-edge vocabulary, service topology, per-framework
entrypoint detection, cross-service reachability. Only then does the
**contract-collision sweep** work: it identifies that microservice analysis and
polyglot analysis **both need boundary-edge vocabulary** (`http` / `rpc` / `queue`),
so that vocabulary is ONE design decision, not two, because the parity clause makes
a term coined twice permanently wrong. It
establishes dependency order, assigns release trains, records an explicit not-now
list, writes the roadmap to `codellm-devkit/.github` → `docs/design/roadmap.md`,
and files **at most one parent issue** — for the single feature actually starting. It
checkpoints before entering `designing-cldk-changes`.

FAIL:
- "microservice static analysis" is carried through as a single roadmap row — a
  theme left undecomposed gives the collision sweep nothing to compare, and the
  boundary-edge overlap becomes invisible;
- four parent issues (or four plus children) filed because the user said "into
  GitHub" — the roadmap is a committed doc, and the tracker gets the one feature
  that is starting;
- the collision between microservice and polyglot analysis is not surfaced, and
  each is treated as an independent design;
- the roadmap is written into an issue body rather than committed as a doc;
- the agent designs all four in depth in this session instead of staying shallow;
- grouping, ordering, or the not-now list is decided solo rather than via
  `AskUserQuestion`;
- the agent auto-invokes `designing-cldk-changes` without a checkpoint;
- the roadmap is tidy and well-ordered but misses the shared boundary-edge
  vocabulary — that alone fails the scenario.
