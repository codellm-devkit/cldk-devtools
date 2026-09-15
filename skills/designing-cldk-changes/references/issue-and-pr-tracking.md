# Issue and pull-request tracking

This file shows how the **Spec → Tracking Record** step of `designing-cldk-changes` becomes a
record on GitHub. The gate is not complete until the spec and its tracking record both exist.

The org-level forms give the text of each issue and each pull request. This file does not give
that text. The forms are in `codellm-devkit/.github`, in `.github/ISSUE_TEMPLATE/` and
`.github/pull_request_template.md`. That repository also holds the conventions, in its
`CONTRIBUTING.md`. Each repository without its own forms gets these forms automatically.

This file gives what the forms cannot. It gives the shape to file, the time to file it, the method
to fill it, and the method to connect the parts.

## The two forms

There are two issue forms. Blank issues are off for the whole org. Therefore one of these two forms
is always in use.

| Form | Use when | Filed on |
| --- | --- | --- |
| [`bug_report.md`][bug] | Something is broken. The output is wrong, the program stops, or a behavior went back. | The repository with the fault |
| [`feature_request.md`][feature] | Everything else. A new function, upkeep, documentation, CI, a refactor, or a migration. | The repository that changes |
| [`pull_request_template.md`][pr] | Every pull request. | Applied automatically |

The org repository owns the shape of a form. This skill owns the time to use it and the reason to
use it. This file does not repeat the forms. A local copy becomes different from the org repository,
and an agent reads the local copy.

> **NOTE:** The `work_item.yml` and `epic.yml` forms were retired on 2026-09-14. Do not cite them.
> Their links give an HTTP 404 error. The three sections that made an issue honest are still
> necessary. The table in **The two sections that hold the retired discipline** shows the new
> position of each one.

## The rule that replaces counting

Tracking granularity follows pull-request granularity. Do not count steps. Do not count
repositories. There is one question: does one pull request close this? If the answer is yes, it is
an issue. If it is a step inside one pull request, it is a checkbox under **Describe the solution
you'd like**.

## Select the shape first, with the user

The shape comes from the **Decomposition and Release Plan** decision in `SKILL.md`. Put that
decision to the user with `AskUserQuestion`. Do not infer the shape from the triage table. There are
three shapes. The smallest shape is first.

| Shape | Use when | Issue count |
| --- | --- | --- |
| **Single issue** | The change lands in one pull request. | 1 |
| **Parent and one sub-issue for each pull request** | The work covers repositories that release on their own clocks. | 1 + pull requests |
| **Parent and a stack of sub-issues** | One rung is very large, such as a full L3/L4 build or a migration with many stages. Its units land as separate pull requests. | 1 + units |

Select the smallest shape that fits. Then let the user make it larger. A backlog that nobody can
read does not keep a design record. It hides one.

Three signs show that the shape is too small:

- A child issue has a body that is one checklist line in a related issue.
- A child issue for documentation holds one sentence for a README file.
- All the rungs are in one repository and land in one pull request, but each rung has a child
  issue.

### Single issue

Rungs become checklist lines under **Describe the solution you'd like**. Rungs do not become
separate issues. Documentation, release and verification become lines in the same section. For a
change of one pull request, this shape is a complete answer to the gate. It is not a method to avoid
the gate.

### Parent shapes

A parent issue is a normal `feature_request` issue with children. There is no separate form for a
parent issue, and the `Epic` label is not necessary.

- The **parent issue** is the record that coordinates the repositories. It holds a short summary, a
  link to the committed spec, the list of affected repositories, the locked design decisions, and
  the release plan. It links the spec. It does not repeat the text of the spec.
- Each **child** is one unit of work that one pull request closes. File each child on the repository
  that it changes. Use the form that fits that child.
- Documentation, release and verification go into the last child. If `docs` is a separate repository
  deliverable with its own pull request, give it its own issue.
- Each child gets a branch. Each branch gets one pull request that closes the child, with
  `Closes #NNN`. When all its sub-issues close, the parent issue closes.

## File at the time you start the work

When you start a unit, open its child issue. When you create the parent issue, do not open the
children. The link to the spec in the parent issue already holds the full plan. The backlog does not
need a copy of that plan.

A plan that becomes many issues at one time becomes inventory. Un-started issues become old, they
hide the live issues, and they make the backlog difficult to read.

The spec and the parent issue complete the gate. The children do not need to exist yet.

## Sub-issues, not checklists

Attach children as native GitHub sub-issues. Sub-issues are a GitHub function, not a function of a
form. The retirement of the epic form did not retire them.

Do not keep a `CHILDREN` checklist by hand. Do not add a `Part of <owner>/<repo>#N` line. GitHub
rolls up a parent and its children automatically. A manual list becomes wrong as soon as something
moves.

To attach a child from the command line, use the `id` of the child. Do not use its number.

```bash
child_id=$(gh api repos/codellm-devkit/<child-repo>/issues/<child-number> --jq .id)
gh api -X POST repos/codellm-devkit/.github/issues/<parent-number>/sub_issues \
  -F sub_issue_id="$child_id"

# verify
gh api repos/codellm-devkit/.github/issues/<parent-number> --jq .sub_issues_summary
```

The child repository and the `.github` repository are different on each parent issue. This is
correct. GitHub permits a parent and a child in different repositories of one org. A `python-sdk`
child and a `codeanalyzer-java` child can attach to the same org parent issue.

## Titles and labels

Write each title as a plain sentence. Do not write a `type(scope):` prefix. Do not write
`chore(templates):`, `feat(cli):`, or `docs(spec):`. A label already gives the type. The prefix also
fills the first 20 characters of each row in a tracker list, and a reader scans that part.

Write each identifier, path, flag and symbol in a title between backticks.

| | |
| --- | --- |
| Wrong | `fix(dataflow): -j 1 and -j N disagree on the file set` |
| Correct | ``Sequential and parallel dataflow disagree on the file set`` with the label `bug` |

The type goes on a label. When you file the issue, apply the label. Use `bug`, `enhancement`,
`documentation` or `chore`, and the other labels of that repository. Use `--label` with
`gh issue create`. Every filter in the tracker ignores an issue with no type label.

## Link the spec. Do not repeat it.

The directories `docs/design/specs/` and `docs/design/plans/` are committed. The parent issue links
the spec. A child links its plan. A copy of a design summary in an issue body made the old parent
bodies difficult to read. A document is also reviewable in a pull request and comparable over time.
An issue body is neither.

## Where each record goes

Put a parent issue that covers more than one repository in `codellm-devkit/.github`. This org
repository already gives the conventions of the org. It holds `CONTRIBUTING.md` and the issue forms.
Do not put such a parent issue on a deliverable repository. Then there is no decision to make and no
example to match.

Two properties make this repository correct. It keeps the tracker of a working repository easy to
read, because the issue list of `codeanalyzer-java` then holds only units of work, one for each pull
request. It is also public, so an outside contributor with a child issue can read the parent issue
and the spec. A private planning repository gives public issues that link to pages their reader
cannot open.

- Put the **parent issue** in `codellm-devkit/.github`.
- Put each **child** on the repository that it changes, such as `codeanalyzer-<lang>`, `python-sdk`
  or `docs`. Attach the child to the parent issue as a cross-repository sub-issue. The limits are
  100 sub-issues for each parent and 8 levels of nesting. At pull-request granularity, neither limit
  is a true constraint.
- Add both to the org project board, **Project 1**, "Codellm-Devkit: Project Planning Board". The
  retired forms held `projects: ["codellm-devkit/1"]` and added themselves. The legacy forms hold
  `projects_v2: codellm-devkit/1`. Make sure that the issue arrived on the board. If the issue is
  not there, add it by hand. The board is the view across repositories. The parent issue is the
  record. Do not change the board by hand for any other reason.

### Where the spec goes

| Spec scope | Committed to |
| --- | --- |
| It touches **one** repository. | The `docs/design/specs/` directory of that repository |
| It touches **more than one** repository. | `codellm-devkit/.github`, in `docs/design/specs/` |

A design that covers more than one repository has no correct position in one of them. A commit to
the analyzer that starts first is arbitrary, and the other four analyzers then link across to it.
Put such a spec with the parent issue that coordinates it.

<HARD-GATE>
CAUTION: File every issue and every pull request on one of the two org forms. Do not file an
approximate copy of a form. Do not file a body with your own headings for the same content.

The commands `gh issue create --body` and `gh pr create --body` go past the form, and they give no
message. If you use one of these commands, copy the sections of the form exactly. Use the same
names, in the same order. Add no section. Remove no section.

If a section does not apply, write the reason that it does not apply. Never remove the section.
</HARD-GATE>

Read the form before you fill it. Do not fill it from memory. Use this command:

```bash
gh api repos/codellm-devkit/.github/contents/.github/ISSUE_TEMPLATE/feature_request.md \
  --jq .content | base64 -d
```

[bug]: https://github.com/codellm-devkit/.github/blob/main/.github/ISSUE_TEMPLATE/bug_report.md
[feature]: https://github.com/codellm-devkit/.github/blob/main/.github/ISSUE_TEMPLATE/feature_request.md
[pr]: https://github.com/codellm-devkit/.github/blob/main/.github/pull_request_template.md

## What a filled-in section looks like

The forms give the sections. This part gives the content for those sections, which is the part that
agents get wrong. It applies to both forms and to the body of a pull request.

Without this guidance, a filled-in issue holds 900 to 1000 words. Those words repeat context, tell
the history of the investigation, and give the same result more than one time. Reviewers then stop.
The risks are the part that makes an issue honest, and reviewers never arrive at that part.

### Budget

Each section has a limit. An issue that needs more is two issues.

**`feature_request`**

| Section | Limit |
| --- | --- |
| Is your feature request related to a problem? | 120 words, in one or two paragraphs |
| Describe the solution you'd like | 8 checkboxes, one line each |
| Describe alternatives you've considered | 40 words, or the scope boundary: what this issue does NOT do |
| Additional context | 5 bullets, 25 words each: risks, unsoundness from a dependency, known gaps |

**`bug_report`**

| Section | Limit |
| --- | --- |
| Describe the bug | 120 words, in one or two paragraphs |
| To Reproduce | 6 numbered steps, one action each |
| Expected behavior | 6 lines: the definition of done, as exact expected sets |
| Logs | Outside the limit. This content is evidence. |
| Additional context | 5 bullets, 25 words each |

| | |
| --- | --- |
| Whole body | 400 words. Code blocks and tables are outside this limit. |

Code blocks, tables and command output are outside the limit. They are evidence. You spend the words
to prevent a second description of that evidence.

The limit binds the structure, not the count. The problem statement is two paragraphs. A third
paragraph is evidence. Put it in a code block, or remove it. A bullet in Additional context is one
sentence. A bullet that needs two sentences is two bullets, or it belongs in the spec.

### The two sections that hold the retired discipline

The legacy forms have no `SCOPE BOUNDARY` section, no `CAVEATS` section and no `DEFINITION OF DONE`
section. Those three sections made an issue honest. Therefore this table gives each one a new
position.

| Retired section | New position |
| --- | --- |
| Scope boundary: what this issue does NOT do | `feature_request`: **Describe alternatives you've considered**. `bug_report`: the first bullet of **Additional context**. |
| Caveats and known risks | **Additional context**, in both forms. |
| Definition of done | `feature_request`: the checkboxes under **Describe the solution you'd like**. `bug_report`: **Expected behavior**. |

A definition of done that says "works correctly" is not a definition of done. Write an exact
expected set. Do not write "non-empty". Write a behavior that you demonstrate. Do not write a
behavior that you assert. Two examples: *the returned set equals the set computed by hand*, and *a
test that fails before the change and passes after it*.

If Additional context repeats the goals with the word "must" in front of each one, that section is
empty. Give the limit of the substrate, the unsoundness from a dependency, or the property that
nobody measured. If there are no risks, write that fact.

### Every claim carries its evidence

A sentence in the problem statement is one of three things. There is no fourth.

1. A fact with a citation. Give `file:line`, or a measured number with the command that produced it.
2. A result that follows from a cited fact. Write it as one clause, in the same sentence.
3. A claim that you did not confirm. Mark it: "unverified:" or "mechanism implies, not tested:".

Show the evidence. Do not describe it. An output block of four lines replaces a paragraph, and a
reader can confirm it:

```
L1  body{"34:15"}  kind=call  callee=null
L2  body{"34:15"}  kind=call  callee="can://…/@external/app.Account/__init__"
```

The third category is necessary. An issue that gives an inference as a measurement sends the next
person to repair something that is not broken.

### Register

Compress with the `caveman-compress` rules. Remove articles, filler, hedges, pleasantries and
connective words. Keep code, paths, commands, identifiers, numbers and headings exact.

Keep the grammar correct. Compression removes words. It does not damage the words that stay. Write
"PyCG spells", not "PyCG say". Write "Two consequences", not "two bad thing". Agreement between
subject and verb costs nothing, and its absence reads as noise in a public tracker.

### Layout

A reader scans an issue two times before a full read. The first scan decides whether to start the
work. The second scan happens during the work. Write for those two scans.

| Element | Rule |
| --- | --- |
| Answer first | The problem statement starts with the fault or the gap. It does not start with the method of discovery. Give the diagnosis after, in one or two sentences. |
| One action for each step | A checkbox is one action. If a checkbox holds two commands, the reader skips the second command. |
| Seven items in view | A section shows a maximum of seven bullets or checkboxes. More than seven is a second issue. |
| Condition before command | Write "If the fixture has no `__init__`, regenerate it". Do not write the reverse. A reader stops at the first word that does not apply. |
| A heading names the action | Add a heading only below a section of the form. Write "Reproduce on Python 3.11", not "Notes". |

Do not put an argument into bullets. The problem statement is prose because it reasons. Fragments
remove the connections, and then the reader must rebuild them. Make the paragraph shorter instead.

### What to leave out

Leave out each section that the form does not request. These five are the most frequent:

- A summary that repeats the problem statement.
- A section for impact, or for the reason the work matters. The team already agreed to the work.
- An implementation proposal. The design belongs in the spec.
- A history of the investigation. This content belongs nowhere.
- A description of the repair. The repair belongs in the pull request.

### Before you file

- Search the existing issues first. If the issue exists, link it. Then write what is new.
- Make sure that the title has no `type(scope):` prefix, and that a label gives the type.

## `gh` commands

File the parent issue one time, at design time. When you start a child, file that child. Do not
file all the children here.

```bash
# 1. The parent issue — ALWAYS in the org `.github` repo, never the deliverable repo.
gh issue create --repo codellm-devkit/.github \
  --title "<plain one-line change, code in backticks>" \
  --label enhancement \
  --body-file /path/to/parent-body.md
# → note the returned number, call it PARENT. The gate is satisfied here:
#   spec committed + parent filed. Children do NOT need to exist yet.

# 2. When you pick up a unit, file its issue on the repo it changes...
gh issue create --repo codellm-devkit/codeanalyzer-<lang> \
  --title "<plain title of the unit closed by one PR>" \
  --label enhancement \
  --body-file /path/to/child-body.md
# → note the returned number, call it CHILD

# 3. ...and attach it across repos as a sub-issue (takes the child's id, NOT its number)
child_id=$(gh api repos/codellm-devkit/codeanalyzer-<lang>/issues/CHILD --jq .id)
gh api -X POST repos/codellm-devkit/.github/issues/PARENT/sub_issues \
  -F sub_issue_id="$child_id"

# 4. Progress rolls up on its own — nothing to tick.
gh api repos/codellm-devkit/.github/issues/PARENT --jq .sub_issues_summary
```

Use `--body-file`. Do not use `--body` with text on the command line. A body of more than one line
stays correct with `--body-file`.

If you file an issue by hand, use the org forms in the GitHub interface. The interface shows each
section. A `--body-file` lets you remove a section without a message.

## Example: native dataflow (L3/L4) for a language

The general shape above comes from the L3/L4 dataflow work. This section gives that work as an
example.

The problem statement of the parent issue says this: add levels 3 and 4 to `codeanalyzer-<lang>`,
with a native CFG, PDG and SDG, and a CPG projection. These levels are the graph substrate for
reachability queries.

The alternatives section holds the line between the provider and the client. This analyzer is only a
graph provider. Slicing and taint are queries of the frontend SDK. They are not functions of the
analyzer.

The additional-context section records the locked substrate choices. These are the CFG source, the
def-use source, the points-to oracle, and the posture on precision.

The large backend rung becomes a stack of pull-request units:

- **L3, intraprocedural, no oracle. Release and tag this level first.** Build the CFG, dominance and
  PDG. Emit `body`, `cfg`, `cdg` and `ddg`. Make the backward-slice gate pass on the fixture. Then
  add parallel fan-out for each callable with `-j`, and test it against `--jobs 1`.
- **L4, interprocedural. This level needs the oracle.** Integrate the oracle, map identity, and
  merge the call graph with provenance. Build summaries with hammock regions and an SCC fixpoint
  with k-limiting. Assemble the SDG with `param_in`, `param_out` and `summary` edges. Replace the
  MVP stub that uses types with propagation that uses points-to information and knows about aliases.
- **CPG Neo4j projection, conformance test and schema bump.** If the Neo4j surface is out of scope,
  omit this unit. The SDG is the core artifact.

The additional-context section also names four risks. These are the risks of the oracle
integration, the unsoundness that the language causes (`eval` and reflection, `cgo` and `unsafe`, or
`setjmp` and `longjmp`), the k-limiting that termination needs, and the rule on parallel
determinism.

The rule on parallel determinism is this: never assign an id during parallel execution, and never
emit during parallel execution. Collect the results. Then sort them by `(signature, node_id)`. The
output of `--jobs N` must be identical, byte for byte, to the output of `--jobs 1`.

The solution checkboxes use exact expected sets. They do not use the word "non-empty". They name
each analyzer gate on the fixture: CFG, dominance, DDG, PDG-slice, summary and SDG. They name the
superset gate `L1 ⊆ … ⊆ L4`. They name a clean Neo4j load with no dangling edge.

Slicing and taint are a separate child on the SDK repository, under `cldk-sdk-frontend`. They are
never pull requests on the analyzer.
