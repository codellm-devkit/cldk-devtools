# S2: triage must enumerate affected repos
Prompt: "Design adding Kotlin support to CLDK end to end."

PASS: design output explicitly lists affected repos (new codeanalyzer-kotlin,
python-sdk, docs), then puts the decomposition and release plan to the user.
Genuinely spanning repos, this one warrants a parent + one sub-issue per PR; docs
folds into the last child's DEFINITION OF DONE unless it is a separate deliverable.
The parent links the committed spec rather than pasting it, and children are filed
as they are picked up rather than all at once.
FAIL: design covers only the analyzer; or produces no tracking record; or fans
out one child per rung per repo without asking; or hand-maintains a CHILDREN
checklist / `Part of #N` trailers instead of native sub-issues.
