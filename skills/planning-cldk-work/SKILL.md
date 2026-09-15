---
name: planning-cldk-work
description: Use when CLDK work is too plural for one design session — a theme that decomposes into several contract decisions, or several initiatives competing for a quarter — before any one of them enters design mode.
---

# Planning CLDK work

The planning mode of the CLDK ladder, and the only mode that reasons across **more
than one contract decision at a time**. Every other mode is single-change scoped:
they take "we have decided to do X" and carry X to release. This one comes before
that — it decides *which* decisions, in what order, and which of them are secretly
the same decision.

Planning is **shallow across many**; design mode is **deep on one**. Resist
designing here.

## Entry Preconditions

**The test: can this be stated as a single contract decision?** If yes, it is design
mode. If no, it is planning. Two shapes fail that test, and both belong here:

- **A theme that decomposes into several contract decisions** — "microservice static
  analysis" is one initiative, still plural. Step 1 of the loop breaks it down.
- **Several initiatives competing for the same quarter.** The portfolio case.

Either way there is something to **order, group, and exclude**.

Counter-test, so the mode does not fire on ordinary work: if you can name the one
contract decision and the only open question is how its PRs sequence, that is
`designing-cldk-changes`'s decomposition step, not a roadmap. Go L3/L4 on one
analyzer is one decision; do not manufacture a roadmap for it.

Vague *wording* is fine and expected — the sweep works on rough shapes. Vague
*altitude* is not: a theme left whole has to be broken into the decisions it
contains before the sweep can compare anything (step 1).

## The Loop

Run it **WITH the user** — every ordering and grouping call is theirs
(`AskUserQuestion`), not a silent pick.

### 1. Inventory — at the altitude of contract decisions

One line per candidate. No design detail.

**A theme is not a candidate — decompose it first.** "Microservice static analysis"
is not one row; it is boundary-edge vocabulary, service topology, per-framework
entrypoints, cross-service reachability — each a contract decision that can be
ordered, deferred, or found to collide with something else. Break themes down to
that altitude before going further, or the collision sweep has nothing to compare.

The test for a row: could it be one design session? If it is clearly several, split
it. If two rows would obviously be decided together, merge them.

### 2. Contract-collision sweep — the reason this mode exists

For each candidate, ask the Contract-Impact Triage question **shallowly**: does it
move schema v2 output (a node/edge kind, field, level, or id shape)? Then
cross-reference the answers:

> **Which candidates touch the same vocabulary?**

Any overlap is **one decision, not two** — it must be settled in a single design
session even if the features ship months apart. The
[parity clause](../designing-cldk-changes/references/canonical-schema.md) forbids
renaming or repurposing shared vocabulary once it exists, so a term coined twice
is coined wrong permanently.

Output: **collision groups**. A group is a set of candidates plus the vocabulary
they share.

### 3. Dependency order

A DAG, not a list. For each candidate: what must exist first? Blocked-by
relationships are the plan — a roadmap that does not say what unblocks what is an
unordered wishlist.

### 4. Release trains

Which train carries which candidate, and where a schema major forces lockstep
between repos. **Candidates sharing a schema major should ride one migration**, not
one migration each — that is usually the largest saving planning finds.

### 5. Explicit not-now

What is deliberately excluded from this pass, and why. A roadmap without a not-now
list is a wishlist; the exclusions are what make the inclusions mean something.

### 6. Commit the roadmap, then start at most one thing

Write the roadmap to **`codellm-devkit/.github` → `docs/design/roadmap.md`**, per
`references/roadmap-template.md`. It is committed and reviewable, alongside the
parent issues and cross-repo specs it coordinates.

Then file a parent issue for **the single feature actually starting now** — and nothing
else. The roadmap records the rest; the tracker does not need to mirror it.

## <HARD-GATE>

No candidate may enter `designing-cldk-changes` until **every theme has been
decomposed to contract-decision altitude AND the collision sweep has run over the
result**. Both halves, or neither counts: a sweep across undecomposed themes runs
clean, reports no groups, and satisfies nothing — the collision is still there, just
invisible.

Designing a feature in isolation that shares vocabulary with another candidate is
the one planning mistake the parity clause makes permanent. Every other planning
error is recoverable by re-planning.

The gate is on decomposition and the sweep, never on the roadmap's length or polish.

## Terminal State

The ONLY skill you invoke after planning-cldk-work is `designing-cldk-changes`, for
the one feature being started.

**Checkpoint first.** Do not auto-invoke it. Summarize the collision groups, the
dependency order, and what was excluded, then `AskUserQuestion` — start the first
feature's design now, start a different one, or stop here (see
`using-cldk-devtools` → Transition Checkpoint).

**Parking after the roadmap is a legitimate outcome, and often the right one.** The
roadmap is the deliverable of this mode; starting a feature is optional and can
happen in a later session without losing anything.

## Red Flags

| Rationalization | Reality |
| --- | --- |
| "File all six parent issues so the board shows the quarter." | Six idle parents is inventory, and inventory rots. File the one you are starting; the roadmap records the rest. |
| "These are separate features, design them separately." | Run the collision sweep first. Shared vocabulary decided twice is decided wrong, permanently. |
| "Put the roadmap in a tracking issue so it's visible." | Docs carry content, trackers carry state. The roadmap is a committed doc; the Project board is the live view. |
| "'Microservice analysis' is one row on the roadmap." | It is a theme, not a decision. Undecomposed, the sweep runs clean and finds nothing — the gate passes and the collision ships anyway. |
| "The user is busy; I'll group and order these myself." | Grouping and ordering are the user's calls. `AskUserQuestion`, never solo. |

## Scope Guard

This mode plans **CLDK feature work**. It is not a general project-management tool:
no estimation, no capacity modelling, no velocity. It reasons about contracts,
dependencies, and release trains — the things specific to a multi-repo analyzer
ecosystem governed by a shared schema.

If a planning session is really a meeting between people, this mode **records and
challenges**; it does not facilitate. Capture what they decide, surface the
collisions they missed, and say plainly when a call is theirs to make.
