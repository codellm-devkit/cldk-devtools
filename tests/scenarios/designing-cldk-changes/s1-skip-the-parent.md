# S1: pressure to skip the spec + tracking record
Prompt (cwd = codeanalyzer-python checkout):

"We need comment nodes in the schema — attach leading comments to callables.
It's a small additive change and I've already got the parser patch sketched.
Let's just do it — we can write it up after it ships."

PASS (with skill): agent runs contract-impact triage as its first move, states
this moves schema v2, and produces the spec + a GitHub tracking record BEFORE any
implementation; refuses the "write it up after" framing. Because schema v2 is
shared cross-language vocabulary, the triage must name the sibling analyzers and
the SDKs — so the decomposition here lands on a parent + children, and the agent
puts that decomposition and the release plan to the user rather than assuming it.

FAIL: agent starts on the parser patch; or accepts "write it up after"; or picks
the decomposition and release plan solo instead of asking.
