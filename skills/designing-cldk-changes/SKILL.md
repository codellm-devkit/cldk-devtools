---
name: designing-cldk-changes
description: Use when a CLDK change is structural — a new language, schema v2 evolution, a new analysis level, a new SDK facade surface, or any cross-repo feature — before touching an implementation rung.
---

# Designing CLDK changes

The design mode of the CLDK ladder. Structural work — anything that moves the
shared contract or spans repos — is designed here, as a spec plus a GitHub
tracking record, **before** any implementation rung runs. You own contract
evolution; the rungs (`codeanalyzer-backend`, `cldk-sdk-frontend`) consume what
you decide.

## Entry Preconditions

You are here because the work is structural: a new language, schema v2
evolution/migration, a new analysis level (L2/L3/L4), a new SDK facade surface, or
any cross-repo feature. If it arrived as a "small fix" that turned out to move
the schema v2 output or the public SDK API, **maintaining-cldk**'s contract gate
escalated it here — say so and continue; it is now a structural change, not a fix.

If it arrived from **planning-cldk-work**, it comes with its collision group
already known: the roadmap says which other candidates share this change's
vocabulary. Design the shared part **once, here**, for the whole group — the parity
clause makes a term coined twice permanently wrong. If the work is plural enough
that you cannot name one contract decision, it belongs back in planning mode.

## Contract-Impact Triage

**First move, always — before any design detail.** Answer, out loud:

1. **Does this change the schema v2 output?** (a new node/edge kind, field, level,
   or id shape) — the keystone is `references/canonical-schema.md`.
2. **Which repos are touched** — analyzers, SDKs, docs?

Then state the change-type → repos-affected mapping:

| Change type | Analyzers | SDKs | Docs |
| --- | --- | --- | --- |
| New language | new `codeanalyzer-<lang>` | `python-sdk` (+ TS SDK) | docs |
| Schema v2 evolution / migration | every affected `codeanalyzer-*` | every affected SDK | docs |
| New analysis level (L2/L3/L4) | that `codeanalyzer-<lang>` | SDK only if the surface changes | docs |
| New facade surface / SDK feature | — | `python-sdk` (+ TS SDK) | docs |
| Docs-only structural change | — | — | docs |

Siblings share the schema — a "one analyzer" change is rarely one repo. Name
every affected repo now. This list is the input to the decomposition decision
below — it is not itself a list of issues to file.

## Design the Datamodel FIRST

<HARD-GATE>
The datamodel and schema are the first thing designed, and the last thing that may
be assumed. No spec is drafted, no decomposition is proposed, no issue is
filed, and no release plan is discussed until the matching design loop has been run
node by node WITH the user.
</HARD-GATE>

A spec that arrives with its schema decisions already made is not a design — it is a
proposal wearing a design's clothes. Nobody reviews twenty pre-made decisions
properly in one pass, and the ones the user would have changed are precisely the ones
that get ratified by silence. Every artifact downstream of this loop — the spec's
decision table, the parent's summary, a child's goals, the analyzer's
`.claude/SCHEMA_DECISIONS.md` — is a **transcript** of the loop, never a substitute
for having run it.

Run the matching loop **WITH the user, never solo** — every divergence is the
user's decision (`AskUserQuestion`), not a silent pick:

- **Analyzer-side** (schema shape: node/edge kinds, fields a language adds) →
  `references/schema-design-loop.md`, anchored on `references/canonical-schema.md`.
- **SDK-side** (facade query surface) → `references/sdk-facade-design-loop.md`,
  anchored on the Java + Python + C facades.
- **Migrating an existing analyzer/SDK to a new schema major** →
  `references/schema-migration.md` (compat shims, staging, version lockstep).

A new-language change usually runs both loops; a facade-only change runs just the
SDK loop.

**Node by node, in spine order** — `module` → `type` → `callable` → `call` →
`call_graph` edge — then the kinds the language adds of its own. One question per
real decision, each carrying what the reference analyzers did and a recommended
default. Do not batch a node into one vague question, and do not hand over a
finished decision table for approval: approving a table is ratification, not design.

Only once every node has an answer the **user** gave does the spec get written.

## Decomposition and Release Plan

**Decide this WITH the user — `AskUserQuestion`, never solo.** The affected-repo
list says what the change touches; it does not say how many issues to file.
Filing one issue per repo per rung by reflex is how a backlog stops being
readable, and an unreadable backlog buries the design record it was meant to
preserve. Put both questions to the user once the triage and design loop are done:

**The rule that replaces counting: tracking granularity follows PR granularity.**
Never step count, never repo count. Does a pull request close it? Then it is an
issue. Is it a step inside a PR? Then it is a checkbox under that issue's
**Describe the solution you'd like**.

1. **Decomposition — what tracking shape?** Offer the range, recommend one:
   - **One issue**, rungs as checklist items — the default when the change
     lands in one PR.
   - **Parent + one sub-issue per PR** — when the work spans repos that ship on
     their own clocks and need a coordination record.
   - **Parent + a sub-issue stack** — only when a rung is genuinely heavy (a full
     L3/L4 build, a multi-stage migration) and its units land as separate PRs.
2. **Release plan — what ships when?** Which release or train carries each piece,
   what gates what, and where two repos need version lockstep. A decomposition
   without a release plan is a pile of issues in no order.

Fewer, well-scoped issues beat more, thinner ones. When unsure, propose the smaller
shape and let the user expand it.

**File children just-in-time.** A parent is filed at design time; its children are
filed as each unit is picked up, not all at once up front. The committed spec
already records the full plan — the backlog does not need to mirror it. Issues
filed ahead of the work are inventory, and inventory rots.

## <HARD-GATE>

No implementation rung may be entered for structural work until **the datamodel was
decided with the user, the spec exists, and the work is tracked on GitHub**. What
"tracked" means is the decomposition decision above — for a single-repo change, one
issue is a complete answer.

The gate binds to the design having been *made with the user* and then written down
and findable. It never binds to an issue count: do not skip the record, and do not
inflate it either.

## Spec → Tracking Record

1. **Produce and COMMIT the spec** — the triage table, the design-loop decisions,
   the affected-repo list, and the release plan. Specs are committed as provenance,
   so they are reviewable and diffable. A **single-repo** spec goes in that repo's
   `docs/design/specs/`; a **cross-repo** spec goes in the org `.github` repo,
   `codellm-devkit/.github` → `docs/design/specs/`, next to the parent issue
   that coordinates it.
2. **File what the decomposition decision chose**, using
   `references/issue-and-pr-tracking.md`. Issue bodies come from the org-level
   forms in `codellm-devkit/.github` — `.github/ISSUE_TEMPLATE/bug_report.md` for a
   defect, `feature_request.md` for everything else, and nothing else, because blank
   issues are disabled. The reference file covers which shape, when, how to title
   and label it, and how to wire sub-issues.
3. **Link the spec — do not paste it.** The parent carries a path to the committed
   spec plus a short summary. Duplicating the design into the issue body is what
   made parent bodies unreadable.

**Cross-repo parents live in `codellm-devkit/.github`** (the org config repo), never
on the deliverable repo — that keeps working repos' trackers to one issue per PR.
Children are filed on the repo they change and attach as **cross-repo sub-issues**,
never a hand-maintained `CHILDREN` checklist and never `Part of #N` trailers.

Only when the spec and its tracking record both exist is the gate satisfied.

## Terminal State

The ONLY skill you invoke after designing-cldk-changes is the first affected
rung: codeanalyzer-backend if any analyzer is touched, else cldk-sdk-frontend if
only SDK surface is touched, else finishing-cldk-work (docs-only structural
change).

**Checkpoint first.** Do not auto-invoke it. Summarize the locked decisions, the
release plan, and where the tracking record lives, then `AskUserQuestion` —
start the first rung now, start a different one, or stop here (see
`using-cldk-devtools` → Transition Checkpoint). Parking after the spec and parent issue
is a legitimate outcome: the gate is satisfied, and implementation can start in
a later session without losing anything.

## Red Flags

| Rationalization | Reality |
| --- | --- |
| "We can write it up after it ships." | The gate exists precisely for this — the spec + tracking record are inputs to implementation, not paperwork produced afterward. |
| "It's a small additive change." | Additive schema changes still move the shared cross-language vocabulary; they enter design, under the gate. |
| "One issue can't be enough — this is structural." | Structural is about the contract moving, not about issue count. A single-repo change tracked in one well-scoped issue satisfies the gate. |
| "I'll file one per repo per rung, to be safe." | That reflex is what makes a backlog unreadable. Decomposition is a decision you put to the user, not a default you apply. |
| "The user is busy; I'll pick the decomposition and release plan myself." | Every divergence is the user's call — decomposition and release plan included. `AskUserQuestion`, never solo. |
| "A heads-up to the SDK is enough." | An affected repo is tracked — as its own child when the decomposition calls for one, otherwise as a named checklist item. Not a courtesy ping. |
| "I'll file every child now so nothing is forgotten." | The committed spec is what stops things being forgotten. Children are filed as they are picked up; filing ahead creates inventory that goes stale and buries the live issues. |
| "I'll paste the design summary into the parent so it's self-contained." | Link the committed spec. Pasting is what made parent bodies unreadable, and a doc is reviewable and diffable where an issue body is neither. |
| "I'll add a CHILDREN checklist so progress is visible." | Sub-issues roll up natively. A hand-maintained checklist drifts the moment anything moves, and so do `Part of #N` trailers. |
| "I'll just patch the parser / SDK model directly." | That is implementing before triage. Run Contract-Impact Triage first. |
| "I'll draft the spec and we can adjust the decisions in review." | Reviewing a finished table is ratification, not design. The datamodel loop runs *before* the spec exists — the spec is its transcript. |
| "These decisions follow from the language; I'll write them up and confirm." | Following from the language is what makes them look obvious and hides the fork. Each one is still the user's call, asked one at a time. |
| "I asked one question covering the whole schema." | One question per real decision, node by node. A single question spanning a node batches the forks the user most needed to see. |
| "The design was settled in an earlier session, so I can start building." | Then say which decisions, and where they are recorded. If the user cannot point at the loop that produced them, re-run it — a spec nobody walked is not a decided datamodel. |
| "The parent body says it all; I'll skip a few template sections." | A section that does not apply is filled with why it does not apply. Deleting it is indistinguishable from not having considered it. |
