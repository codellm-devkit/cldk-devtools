# Issue and pull-request tracking

This is how the **Spec → Tracking Record** step of `designing-cldk-changes` materializes on GitHub.
The gate is not satisfied until both the spec **and** its tracking record exist.

**Issue and pull-request bodies come from the org-level templates, never from this file** —
`codellm-devkit/.github` → `.github/ISSUE_TEMPLATE/` and `.github/pull_request_template.md`, with
the convention written up in that repo's `CONTRIBUTING.md`. Every repo without its own templates
picks them up automatically. This file covers what the templates cannot: **which shape to file,
when to file it, how to fill it, and how to wire the pieces together.**

## The two forms

There are exactly two issue forms, and blank issues are disabled org-wide, so one of them is always
used.

| Form | Use when | Filed on |
| --- | --- | --- |
| [`bug_report.md`][bug] | something is broken — wrong output, a crash, a regression | the repo it affects |
| [`feature_request.md`][feature] | everything else — new capability, upkeep, docs, CI, refactor, migration | the repo it changes |
| [`pull_request_template.md`][pr] | every pull request | applied automatically on open |

**The org repo owns the shape; this skill owns when and why.** The forms are deliberately not
reproduced here. A local copy drifts from the org repo, and the copy is what an agent reads.

> The `work_item.yml` and `epic.yml` forms were retired on 2026-09-14. Do not cite them; they 404.
> The discipline they enforced — a scope boundary, honest caveats, an exact definition of done —
> did not retire with them. It now lives inside the two sections named below.

## The rule that replaces counting

**Tracking granularity follows PR granularity — never step count, never repo count.** The only
question is: *does a pull request close this?* If yes, it is an issue. If it is a step inside a PR,
it is a checkbox under **Describe the solution you'd like**.

## Pick the shape first — with the user

The shape comes from the **Decomposition and Release Plan** decision in `SKILL.md`, which is put to
the user with `AskUserQuestion`. It is not inferred from the triage table. There are three shapes,
smallest first:

| Shape | Use when | Issue count |
| --- | --- | --- |
| **Single issue** | the change lands in one PR | 1 |
| **Parent + one sub-issue per PR** | the work spans repos that ship on their own clocks | 1 + PRs |
| **Parent + a sub-issue stack** | a rung is genuinely heavy (full L3/L4 build, multi-stage migration) and its units land as separate PRs | 1 + units |

**Default to the smallest shape that fits, and let the user expand it.** A backlog nobody can read
does not preserve a design record — it buries one. Signals that you have gone too fine: a child
issue whose whole body would be one checklist line in its sibling; a "docs" child that is one
sentence appended to a README; a child per rung when all the rungs are in one repo and land in one
PR.

### Single issue

Rungs become checklist lines under **Describe the solution you'd like**, not separate issues; docs
and release/verify become lines in the same section. This is a complete answer to the gate for a
one-PR change — it is not a shortcut around it.

### Parent shapes

A parent is an ordinary `feature_request` issue that happens to have children. There is no separate
form for it, and no `Epic` label is required.

- **The parent** is the cross-repo coordination record. It holds a short summary, a **link to the
  committed spec** (not a paste of it), the affected-repo list, the locked design decisions, and the
  release plan — all inside the four sections the form gives it.
- **Each child** is a single unit of work closed by a single PR, filed on the repo it changes with
  whichever of the two forms fits it.
- Docs, release, and verify fold into the last child unless `docs` is a separate repo deliverable
  with its own PR — then it earns its own issue.
- **Each child → a branch → one PR that closes it** (`Closes #NNN`). The parent closes when its
  sub-issues do.

## File just-in-time

**Open a child when you pick up that unit, not when the parent is created.** The parent's spec link
already records the full plan; the backlog does not need to mirror it. Filing every future unit up
front converts a plan into inventory — un-started issues go stale, bury the live ones, and make the
backlog unreadable.

The gate is satisfied by the spec plus the parent issue. It does not require the children to exist
yet.

## Sub-issues, not checklists

Children are attached as **native GitHub sub-issues**. Sub-issues are a GitHub mechanism, not a
template feature — retiring the epic form did not retire them. Do **not** hand-maintain a `CHILDREN`
checklist and do **not** add `Part of <owner>/<repo>#N` trailers; GitHub does parent/child rollup
natively, and the manual forms drift the moment anything moves.

Attaching from the CLI takes the child's **`id`**, not its number:

```bash
child_id=$(gh api repos/codellm-devkit/<child-repo>/issues/<child-number> --jq .id)
gh api -X POST repos/codellm-devkit/.github/issues/<parent-number>/sub_issues \
  -F sub_issue_id="$child_id"

# verify
gh api repos/codellm-devkit/.github/issues/<parent-number> --jq .sub_issues_summary
```

`<child-repo>` and the `.github` repo differ on every parent — that is the point, and GitHub allows
a parent and child to live in different repositories within an org. A `python-sdk` child and a
`codeanalyzer-java` child hang off the same org parent.

## Titles and labels

**Titles are plain sentences.** No `type(scope):` prefix — no `chore(templates):`, no
`feat(cli):`, no `docs(spec):`. The prefix duplicates what a label already encodes and eats the
first twenty characters of every row in a tracker list, which is the part a reader scans.

Identifiers, paths, flags and symbols inside a title go in `` `backticks` ``.

| | |
| --- | --- |
| Bad | `fix(dataflow): -j 1 and -j N disagree on the file set` |
| Good | ``Sequential and parallel dataflow disagree on the file set`` + label `bug` |

**The type lives in a label**, applied at file time: `bug`, `enhancement`, `documentation`, `chore`,
plus whatever the repo already uses. Set it with `--label` on `gh issue create`; a typeless issue is
invisible to every filter the tracker offers.

## Provenance: link the spec, don't paste it

`docs/design/specs/` and `docs/design/plans/` are **committed**. The parent links the spec it came
from; a child links its plan. Duplicating a design summary into an issue body is what made parent
bodies unreadable — and a doc is reviewable in a PR and diffable over time, which an issue body is
not.

## Placement convention

**Cross-repo parents live in one place: `codellm-devkit/.github`**, the org repo that already
defines how the org works (it holds `CONTRIBUTING.md` and the issue forms). Not on the deliverable
repo — there is no judgement call to make and no precedent to match.

Two properties make this the right home. It keeps a working repo's tracker readable —
`codeanalyzer-java`'s issue list then contains only units of work, one per PR. And it is **public**,
so an outside contributor picking up a child can still read the parent that explains it and the spec
it links to. A private planning repo would leave public issues referencing things their reader
cannot open.

- **Parent** → `codellm-devkit/.github`.
- **Each child** → the repo it changes (`codeanalyzer-<lang>`, `python-sdk`, `docs`, …), attached to
  the parent as a **cross-repo sub-issue**. Limits: 100 sub-issues per parent, 8 levels of nesting —
  neither is a real constraint at PR granularity.
- Add both to the org project board (**Project 1**, "Codellm-Devkit: Project Planning Board"). The
  retired forms declared `projects: ["codellm-devkit/1"]` and added themselves; the legacy forms
  carry `projects_v2: codellm-devkit/1`, so verify the issue landed and add it by hand if it did
  not. The board is the cross-repo *view*; the parent is the cross-repo *record*. Do not hand-curate
  it beyond that.

### Where the spec goes

| Spec scope | Committed to |
| --- | --- |
| Touches **one** repo | that repo's `docs/design/specs/` |
| Touches **several** repos | `codellm-devkit/.github` → `docs/design/specs/` |

A cross-repo design has no natural home in any one of the repos it changes — committing it to
whichever analyzer happened to go first is arbitrary, and the other four then link sideways into it.
Put it with the parent that coordinates it.

<HARD-GATE>
Every issue and every pull request is filed on one of the two org forms — not an approximation of
one, and not a body with your own headings covering the same ground.

`gh issue create --body` and `gh pr create --body` bypass the form silently. When you use them,
reproduce the form's sections EXACTLY: same names, same order, none added, none dropped.

A section that does not apply is filled with the reason it does not apply. It is never deleted.
</HARD-GATE>

Read the form before filling it, rather than recalling it:

```bash
gh api repos/codellm-devkit/.github/contents/.github/ISSUE_TEMPLATE/feature_request.md \
  --jq .content | base64 -d
```

[bug]: https://github.com/codellm-devkit/.github/blob/main/.github/ISSUE_TEMPLATE/bug_report.md
[feature]: https://github.com/codellm-devkit/.github/blob/main/.github/ISSUE_TEMPLATE/feature_request.md
[pr]: https://github.com/codellm-devkit/.github/blob/main/.github/pull_request_template.md

## What a filled-in section looks like

The forms give the slots. This gives the shape of what goes in them — the part agents get wrong. It
applies to both forms above, and to pull-request bodies. Left alone, a filled-in issue runs 900–1000
words of restated context, narrated investigation and repeated findings. Reviewers stop reading, and
the risks — the part that makes the issue honest — are what they never reach.

### Budget

Each section is capped. An issue that needs more is two issues.

**`feature_request`**

| Section | Budget |
| --- | --- |
| Is your feature request related to a problem? | 120 words, one or two paragraphs |
| Describe the solution you'd like | 8 checkboxes, one line each |
| Describe alternatives you've considered | 40 words — or the scope boundary: what this explicitly does NOT do |
| Additional context | 5 bullets, 25 words each — risks, inherited unsoundness, known gaps |

**`bug_report`**

| Section | Budget |
| --- | --- |
| Describe the bug | 120 words, one or two paragraphs |
| To Reproduce | 6 numbered steps, one action each |
| Expected behavior | 6 lines — the definition of done, as exact expected sets |
| Logs | outside the budget; evidence |
| Additional context | 5 bullets, 25 words each |

| | |
| --- | --- |
| Whole body | 400 words, excluding code blocks and tables |

Code blocks, tables and command output are outside the budget. They are evidence, and evidence is
what the words are being spent to avoid restating.

The budget binds structurally, not by counting: **the problem statement is two paragraphs.** A third
paragraph is evidence — put it in a code block or delete it. **An Additional-context bullet is one
sentence.** A bullet needing two sentences is two bullets, or it belongs in the spec.

### The two sections that carry the retired discipline

The legacy forms have no `SCOPE BOUNDARY`, `CAVEATS` or `DEFINITION OF DONE` slot. Those three did
the work of making an issue honest, so they are placed deliberately rather than dropped:

| Retired section | Now lives in |
| --- | --- |
| Scope boundary — what this does NOT do | `feature_request` → **Describe alternatives you've considered**; `bug_report` → first bullet of **Additional context** |
| Caveats and known risks | **Additional context**, both forms |
| Definition of done | `feature_request` → checkboxes under **Describe the solution you'd like**; `bug_report` → **Expected behavior** |

**A definition of done that says "works correctly" is not one.** Prefer an exact expected set over
"non-empty", and a demonstrated behaviour over an asserted one: *the returned set equals the
hand-computed one*, *a test that fails before and passes after*.

**If Additional context is a restatement of the goals with "must" in front, it is empty** — name the
substrate limit, the inherited unsoundness, or the thing that is unmeasured. If there genuinely are
no risks, say so explicitly.

### Every claim carries its receipt

A sentence in the problem statement is one of three things, and nothing else:

1. **A fact with a citation** — `file:line`, or a measured number with the command that produced it.
2. **A consequence that follows from a cited fact** — one clause, in the same sentence.
3. **A claim you have not verified, marked as such** — "unverified:", "mechanism implies, not tested:".

Prefer showing to describing. A four-line output block replaces a paragraph and is checkable:

```
L1  body{"34:15"}  kind=call  callee=null
L2  body{"34:15"}  kind=call  callee="can://…/@external/app.Account/__init__"
```

The third category is not optional politeness. An issue that states an inference as a measurement
sends the next person to fix something that is not broken.

### Register

Compress with the `caveman-compress` rules — drop articles, filler, hedging, pleasantries and
connective fluff; preserve code, paths, commands, identifiers, numbers and headings exactly.

**Grammar stays correct.** Compression means deleting words, never mangling the ones that remain.
"PyCG spells", not "PyCG say". "Two consequences", not "two bad thing". Subject-verb agreement and
plurals cost nothing and their absence reads as noise in a public tracker.

### Layout

An issue gets scanned twice before it is read: once by whoever decides to pick it up, once by
whoever does. Arrange for the scan.

| Element | Rule |
| --- | --- |
| Answer first | The problem statement opens with what is broken or missing, never with how you found it. Diagnosis after, in a sentence or two. |
| One action per step | A checkbox is one action. Two commands in one box means the second gets skipped. |
| Seven items in view | No section shows more than seven bullets or boxes. More than that is a second issue. |
| Condition before command | "If the fixture has no `__init__`, regenerate it" — never the reverse. A reader stops at the first word that does not apply to them. |
| Headings name what happens there | Only where you add one beneath a form section. "Reproduce on Python 3.11", not "Notes". |

**Do not bullet an argument.** The problem statement is prose because it reasons; chopping it into
fragments to look skimmable strips the connective tissue and leaves the reader to reassemble it.
Shorten the paragraph instead.

### Leave out

Sections the template does not ask for. Most often: a summary that repeats the problem statement, an
"impact"/"why it matters" section arguing for work already agreed, a proposed implementation, and a
narration of how the investigation went. The design belongs in the spec; the fix belongs in the PR;
the search path belongs nowhere.

### Before filing

- Search existing issues first and link the duplicate instead of filing. Say what is new.
- Check the title carries no `type(scope):` prefix and the type is on a label instead.

## `gh` invocations

The parent is filed once, at design time. Children are filed **as each is picked up** — not all at
once here.

```bash
# 1. The parent — ALWAYS in the org `.github` repo, never the deliverable repo.
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

Use `--body-file` (not inline `--body`) so multi-line bodies survive intact.

When filing interactively rather than from a script, prefer the org issue forms in the GitHub UI —
they present the sections a `--body-file` lets you quietly omit.

## Worked example — native dataflow (L3/L4) for a language

The generalized form above is the distillation of the concrete L3/L4 dataflow effort. Instantiated,
its **parent** problem statement says "add levels 3–4 (native CFG/PDG/SDG + CPG projection) to
`codeanalyzer-<lang>` as the graph substrate reachability queries run over"; its **alternatives**
section carries the provider/client line ("this analyzer is a pure graph provider — slicing and
taint are frontend SDK queries, not analyzer features"); its **additional context** records the
locked substrate choices (CFG source, def-use source, points-to oracle, precision posture); and its
heavy backend rung fans into a PR-unit stack:

- **L3 (intraprocedural, no oracle — ship and tag first):** CFG + dominance + PDG,
  `body`/`cfg`/`cdg`/`ddg` emission, the backward-slice gate green on the fixture, then per-callable
  parallel fan-out (`-j`) differential-tested against `--jobs 1`.
- **L4 (interprocedural — needs the oracle):** oracle integration + identity mapping + call-graph
  merge with provenance; summaries (hammock regions, SCC fixpoint with k-limiting); SDG assembly
  with `param_in`/`param_out`/`summary` edges; points-to-backed (alias-aware) propagation replacing
  the type-based MVP stub.
- **CPG Neo4j projection + conformance test + schema bump** (skip if the Neo4j surface is out of
  scope; the SDG is the core artifact).

Its **additional context** names the oracle-integration risks, the inherited unsoundness for the
language (eval/reflection | cgo/unsafe | setjmp-longjmp), the k-limiting-for-termination
requirement, and the parallel-determinism rule (never assign ids or emit during parallel execution —
collect, then sort by `(signature, node_id)`; `--jobs N` byte-identical to `--jobs 1`). Its
**solution checkboxes** use exact expected sets, not "non-empty": every analyzer gate on the fixture
(CFG, dominance, DDG, PDG-slice, summary, SDG), the `L1 ⊆ … ⊆ L4` superset gate, and a clean Neo4j
load with no dangling edges. Slicing + taint are a **separate child on the SDK repo**
(`cldk-sdk-frontend`), never PRs on the analyzer.
