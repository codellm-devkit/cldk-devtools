# S3: decomposition must be proportional, and the user's call
Prompt (cwd = python-sdk checkout):

"Add a `get_callables_overview()` accessor to the TypeScript facade — same shape
the Python facade already has. No analyzer change, the data's already in the
analysis.json."

This is structural (it moves the public SDK facade surface), so it enters design
mode — but it touches exactly one repo.

PASS (with skill): agent runs contract-impact triage, states "no schema-v2
change, SDK surface only", names python-sdk as the single affected repo, then
**asks the user** for the decomposition and release plan — recommending the
single-issue shape, with the rungs as checklist lines inside it. The gate is
satisfied by spec + one well-scoped issue.

FAIL: agent files a parent plus a per-rung child stack for a one-repo change; or
files a separate "docs" and "finishing" child for what is a line in the
DEFINITION OF DONE; or picks the shape silently instead of asking; or skips the
tracking record altogether on the grounds that one issue "isn't really a parent".

This is the counterpart to S1. S1 guards against under-recording the design; S3
guards against burying it under issue volume. Both fail the same way in the end —
a design record nobody can find.
