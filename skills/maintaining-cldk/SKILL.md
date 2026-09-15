---
name: maintaining-cldk
description: Use when picking up an issue, bug report, small feature, or documentation gap on any codellm-devkit repository, or when triaging whether a reported problem is real.
---

# Maintaining CLDK

The upkeep mode of the CLDK ladder: triage → contract gate → fix loop →
propagation sweep. This is where bug fixes, small features, and docs gaps
land — the counterpart to `designing-cldk-changes`, which owns anything
structural. Most work enters here.

## Entry Preconditions

An issue, bug report, small feature request, or docs gap already exists. No
spec and no parent issue are needed to start — that is the point of this mode. If
what you're holding turns out to need a design decision spanning repos, the
contract gate below routes you out.

## 1. Triage

**Reproduce first, always** — do not start editing on the strength of a
report alone:

- Analyzer bug → build a minimal fixture that exhibits it.
- SDK bug → write the failing test first.
- Docs gap → reproduce the broken render or the broken command.

Locate the affected repo(s) via `references/repo-map.md`. Classify what
you're holding. Triage may terminate right here with a **triage verdict**:
not-a-bug, duplicate, or needs-design (route to `designing-cldk-changes`).
If it's real and local, continue to the gate below.

<HARD-GATE>
If the fix changes schema v2 output (any node/edge/field, even optional
additions) or a public SDK API, STOP. This is structural work and cannot be
done here. 'Optional so it can't break anyone' is the canonical
rationalization — additive fields are still contract changes.

Report the verdict plainly, then CHECKPOINT before moving: `AskUserQuestion`
whether to enter designing-cldk-changes now, park it as an issue for later, or
stop. The verdict is not negotiable; the timing is the user's. Never offer
"patch it here anyway" as an option.
</HARD-GATE>

## 2. Fix Loop

Failing test first, then the fix. Follow the target repo's own `CLAUDE.md`
conventions (branch naming, commit style, test layout) — this mode does not
override local repo discipline, it sits on top of it.

**Everything you file goes on an org form** — the bug report, the feature
request, and the PR that closes them. Not a hand-rolled body covering the same
ground. The forms and the rules for filling them are in
`designing-cldk-changes` → `references/issue-and-pr-tracking.md` § The
forms; the PR body specifically is in `finishing-cldk-work` § Pull Request
Body. `gh issue create --body` and `gh pr create --body` bypass the form
silently — reproduce its sections exactly, or pass `--template`.

## 3. Propagation Sweep

Before you consider the work done, run `references/propagation-checklist.md`
— sibling analyzers, SDK version pins, docs staleness, and old-behavior
fixtures elsewhere. This produces a **required** output, in this exact shape:

**Propagation verdict:** <list of follow-on repos + why> | none, because
<reason>.

Skipping this step is the second canonical failure mode of this mode, not a
lesser one than skipping the contract gate.

## Terminal State

The ONLY skill you invoke after maintaining-cldk is finishing-cldk-work (or
designing-cldk-changes via the contract gate; or stop at a triage verdict if
there is nothing to fix).

**Checkpoint first.** Do not auto-invoke it. State what you finished, name the
next skill and why, then `AskUserQuestion` — proceed now, take a different path,
or stop here (see `using-cldk-devtools` → Transition Checkpoint). When the
contract gate fired, report the verdict plainly *and still ask* whether to enter
design mode now; the verdict is not up for negotiation, the timing is.

## Red Flags

| Rationalization | Reality |
| --- | --- |
| "It's optional, can't break anyone." | Additive schema/API surface is still a contract change — the HARD-GATE applies regardless of optionality. |
| "Tests pass, wrap up." | Local tests passing is not the propagation sweep. Run `propagation-checklist.md` and produce the verdict before declaring done. |
| "That's a scope call I can just make in this conversation." | A schema-shape question is not a same-turn negotiation with whoever asked — it leaves this rung entirely, via the contract gate. |
| "This bug is local to this repo, no need to check siblings." | Siblings share the schema and the resolver patterns; a bug class rarely respects repo boundaries. The sweep exists precisely to check. |
| "I'll write a clear issue body; the form is boilerplate." | The form IS the format. A clear body with your own headings is still off-template, and the sections you would have skipped are the ones that make it honest. |
| "`gh issue create --body` is faster than the browser." | It is, and it bypasses the form silently. Reproduce the sections exactly or pass `--template`. |
