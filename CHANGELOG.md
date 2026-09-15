# Changelog

All notable changes to the CLDK DevTools plugin are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.6.0] — 2026-09-14

### ⚠️ Changed — breaking

The `work_item` and `epic` issue forms are retired. `codellm-devkit/.github` now ships
`bug_report.md` and `feature_request.md` only, with `blank_issues_enabled: false` so one of
the two is always used (`codellm-devkit/.github#87`).

- **`designing-cldk-changes`** → `references/epic-and-issue-templates.md` is renamed
  `references/issue-and-pr-tracking.md` and rewritten against the two surviving forms. The
  three tracking shapes survive with the epic renamed to a **parent issue** — an ordinary
  `feature_request` that happens to have children. Native cross-repo sub-issues are unchanged;
  they are a GitHub mechanism, not a template feature.

- **The discipline the retired forms enforced is placed, not dropped.** A new table maps each
  retired section to where it now lives: scope boundary → *Describe alternatives you've
  considered*; caveats and known risks → *Additional context*; definition of done → the
  checkboxes under *Describe the solution you'd like*, or *Expected behavior* on a bug. The
  per-section word budget is re-cut against the legacy section names.

- **Titles are plain sentences, types are labels.** A new **Titles and labels** section bans
  `type(scope):` prefixes on issues, pull requests and commit subjects — the prefix duplicates
  what a label encodes and eats the first twenty characters of every row a reader scans.
  Identifiers in a title go in backticks. `gh issue create` carries `--label` instead.

- **`planning-cldk-work`, `maintaining-cldk`, `finishing-cldk-work`, `codeanalyzer-backend`,
  `cldk-sdk-frontend`, `using-cldk-devtools`** and the schema/facade design-loop references
  follow the same rename. No ladder rung, gate or routing rule changed.

### Changed

- **Decision records move into `CLAUDE.md`.** Every skill that told a session to write schema or
  facade decisions to `.claude/SCHEMA_DECISIONS.md` / `.claude/FACADE_DECISIONS.md` now points at a
  **Schema decisions** / **Facade decisions** heading in the repo's own `CLAUDE.md`. A dot-directory is
  hidden by common global ignores and read by one tool; `CLAUDE.md` is repository content every agent
  loads. Existing repos fold the old file into `CLAUDE.md` and delete `.claude/`
  (`codeanalyzer-iac#13` is the reference change).

## [0.5.1] — 2026-09-14

### ⚠️ Changed — behavioural

Every issue and pull request is filed on an org form.

- **`designing-cldk-changes`** → `references/epic-and-issue-templates.md` no longer
  reproduces the Epic and Work-item forms. It links them instead, on the same
  principle as the roadmap template in 0.4.1 — the org repo owns the shape, this
  skill owns when and why. A local copy drifts, and the copy is what an agent reads.
  The table now covers all five forms, including `bug_report`, `feature_request`
  and the pull-request template, under a hard gate: reproduce the sections exactly
  or pass `--template`; a section that does not apply is filled with why, never
  deleted. `gh issue create --body` and `gh pr create --body` bypass the form
  silently, and that is now stated where agents will hit it.

- **`finishing-cldk-work`** gains a **Pull Request Body** section — the first PR-body
  guidance in the ladder. It binds the PR to the org template and to the same prose
  discipline as a work item: link the issue rather than restating it, paste the test
  run rather than asserting it, name the contract that moved or write "none".

- **`maintaining-cldk`** carries the rule into the fix loop, where bug reports and
  feature requests are actually filed.

- The work-item prose guidance gains a **Layout** subsection: answer first, one action
  per checkbox, at most seven items in view, condition before command, and no
  bulleting of an argument that needs to reason.

## [0.5.0] — 2026-08-07

### ⚠️ Changed — behavioural

Design mode now leads with the datamodel, and the gate says so.

- **`designing-cldk-changes`** replaces its passive "Design Loops" section with
  **Design the Datamodel FIRST**, under its own hard gate: no spec drafted, no
  decomposition proposed, no epic or issue filed, and no release plan discussed
  until the matching design loop has been run node by node with the user. The loop
  walks the spine in order — `module` → `type` → `callable` → `call` →
  `call_graph` edge — one question per real decision.

  The old ordering let a spec arrive with its schema decisions already made and
  merely presented for approval. Nobody reviews twenty pre-made decisions properly
  in one pass, and the ones worth changing are exactly the ones ratified by
  silence. Every downstream artifact — the spec's decision table, the epic summary,
  `.claude/SCHEMA_DECISIONS.md` — is now stated to be a *transcript* of the loop
  rather than a substitute for having run it.

- The main hard gate binds to the datamodel having been **decided with the user**,
  not just to a spec existing on disk.

- **`codeanalyzer-backend`** gains the matching entry precondition: a spec that
  predates the session must have its locked schema decisions named and re-confirmed
  before scaffolding. A spec nobody walked sends the work back to design mode.

- Four red flags added for the rationalizations that produce a pre-decided spec:
  drafting first and adjusting in review, decisions that "follow from the
  language", one question covering a whole node, and treating an earlier session's
  spec as settled.

- `README.md` is brought back in step with the skills: its ladder diagram carried the
  pre-0.5.0 caption, and its `codeanalyzer-backend` entry still gated on a spec alone.
  The diagram's right-hand column was also three characters out of true, in both the
  README and the dispatcher skill; it is square again.

## [0.4.1] — 2026-08-06

### 🔧 Changed

- The roadmap skeleton moves to `codellm-devkit/.github` →
  `docs/design/roadmap-template.md`, beside the issue forms. It was prose in a fenced
  block inside this skill's reference — not copyable, not where the artifact lands,
  and carrying editorial voice into every roadmap made from it. The reference now
  links the template and keeps only the skill-side rules.

## [0.4.0] — 2026-08-04

### ✨ Added

- **`planning-cldk-work`** — a new mode upstream of design, for work that cannot be
  stated as a single contract decision: a theme that decomposes into several
  (e.g. "microservice static analysis"), or several initiatives competing for a
  quarter. Its core is the **contract-collision sweep**, which finds candidates
  sharing schema vocabulary before either enters design — the parity clause makes a
  term coined twice permanently wrong, and no other mode sees more than one change
  at a time. Produces a committed roadmap plus an epic for the one decision
  starting, never one epic per candidate.
- `references/roadmap-template.md`, and scenarios `s1`–`s3` covering the collision
  sweep, a single decision dressed in planning language, and a single theme that is
  plural underneath.

## [0.3.0] — 2026-08-03

### ⚠️ Changed — **BREAKING** (behavioural)

This release changes how agents decompose and track work. Sessions running under
0.2.0 conventions will behave differently after updating; anyone relying on the old
shape should read this before upgrading.

- **Issue decomposition is now proportional and user-decided.** 0.2.0 mandated an
  epic plus one child per ladder rung for any structural change touching ≥1 rung,
  and its Red Flags table explicitly forbade scaling that down. Tracking granularity
  now follows **PR granularity** — one issue per pull request — and the shape is put
  to the user rather than applied by default. A single-PR change is one issue.
- **`<HARD-GATE>` semantics changed.** It previously bound to "the spec AND the
  GitHub epic + child issues exist". It now binds to "the spec exists and the work
  is tracked", never to an issue count.
- **Removed two Red Flags rows** that forbade smaller tracking shapes
  ("Scale the writing, never the gate"; "any structural change that touches ≥1 rung
  gets an epic + one child per rung").
- **Epics moved repos.** They now live in `codellm-devkit/.github`, not on the repo
  owning the deliverable.
- **`Part of #N` trailers and hand-maintained `CHILDREN` checklists are retired** in
  favour of native GitHub sub-issues. Existing epics carrying either will not be
  updated automatically.
- **Specs and plans moved** from `docs/superpowers/` to a tool-neutral
  `docs/design/{specs,plans}/`, and are committed as provenance rather than
  gitignored scratch.
- **Every ladder transition now stops for the user.** All nine transition points
  — forward, and the backward/sideways gate escalations — announce and ask before
  invoking the next skill. An end-to-end run is materially more interactive than
  under 0.2.0.

### ✨ Added

- Issue bodies now come from org-level forms in `codellm-devkit/.github`
  (`epic.yml`, `work_item.yml`), with the convention in that repo's
  `CONTRIBUTING.md`.
- Scenario `s3-proportional-decomposition`, covering over-decomposition of a
  single-repo change.
