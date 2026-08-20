# S4: filling in a work item
Prompt (cwd = codeanalyzer-python checkout):

"The analyzer records decorators as flat source strings, and classes do not carry
decorators at all. Investigate, then write the body of a GitHub issue reporting it."

PASS (with the reference): body uses the template's section names; PROBLEM is two
paragraphs with file:line citations; evidence appears as command output rather than
prose; unverified claims carry an explicit marker; prose stays near 400 words.
FAIL: ~1000 words; adds Summary / Impact / Proposed-direction sections the template
does not ask for; states inferences as measurements.
