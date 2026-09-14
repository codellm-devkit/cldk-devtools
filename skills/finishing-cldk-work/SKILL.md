---
name: finishing-cldk-work
description: Use when implementation on a CLDK branch is complete and the work needs verification, merge, release, documentation updates, and issue closeout — before claiming any CLDK work is done.
---

# Finishing CLDK work

The ladder's exit. Every other rung — `designing-cldk-changes`,
`maintaining-cldk`, `codeanalyzer-backend`, `cldk-sdk-frontend` — terminates
here. Nothing is "done" until it has passed through this gate: gates green,
a real ship decision, release mechanics if warranted, and closeout.

## Entry Preconditions

Implementation is complete on a branch. Most arrivals come from
`maintaining-cldk`, carrying a **propagation verdict** in hand (the
required output of its propagation sweep — a list of follow-on repos, or
`none` with its reasoning). Arrivals from `codeanalyzer-backend` or
`cldk-sdk-frontend` carry no verdict of their own; treat their Terminal
State handoff as "implementation complete, gates not yet re-run here."
Either way, do not start from a bare "it's done" claim with no artifact
behind it.

## Verification Gates

Which commands to run depend on the repo type — analyzer, SDK, or docs —
see the full matrix in `references/release-gates.md`.

<HARD-GATE>
No success claim, no tag, no release, no 'done' without running the gate
commands NOW and reading their output. 'Passed earlier' is not evidence.
</HARD-GATE>

A same-day CI run, a teammate's "LGTM," or your own memory of an earlier
pass are all the same failure shape: assurance detached from a command you
just watched execute. Re-run the gate on the exact commit you're about to
act on.

## Pull Request Body

The body comes from the org template — `codellm-devkit/.github` →
[`pull_request_template.md`][pr-template]. Opening a PR in the browser applies it automatically.

<HARD-GATE>
`gh pr create --body` bypasses the template silently. When you use it, reproduce the template's
sections exactly — same names, same order, none added, none dropped — or pass `--template`.
A section that does not apply is filled with the reason it does not apply, never deleted.
</HARD-GATE>

Fill it under the same discipline as a work item (`designing-cldk-changes` →
`references/epic-and-issue-templates.md`): evidence over description, every claim carrying its
receipt, 400 words outside code blocks.

- **Motivation and context** — link the issue (`Closes #NNN`); do not restate it. A body that
  duplicates its issue at the same length is adding nothing.
- **How has this been tested?** — the command and its output. "Tested locally" is not an answer;
  a pasted failing-then-passing run is.
- **Breaking changes** — name the contract that moved, or write "none" explicitly. Silence reads
  as not considered.
- **Checklist** — ticked from fact. An unticked box with a one-line reason beats a ticked box
  that is not true.

[pr-template]: https://github.com/codellm-devkit/.github/blob/main/.github/pull_request_template.md

## Ship Decision

Passing gates only tells you the branch is mergeable — it does not tell you
whether to release. Decide explicitly:

- **Forces a release:** a behavior change reaches users (a bug fix, a new
  field, a new accessor), or a downstream pin needs to move (an SDK's
  `codeanalyzer-<lang>==X` needs the fix that just landed).
- **Merge-only, no release:** docs-only changes, internal refactors with no
  observable behavior change, test-only changes. Merge and stop — cutting a
  release train for these is manufacturing release noise, not shipping
  value. The one exception: if docs are themselves published from a tagged
  release (a docs-site build keyed to a version), a "docs-only" change can
  still need one — check before assuming not.

## Release Mechanics

Once a release is warranted, mechanics differ by repo type — full detail in
`references/packaging-and-release.md`:

- **Analyzer** (`codeanalyzer-<lang>`) — a tag-triggered pipeline: PyPI thin
  wheel, GitHub Release binaries, and a Homebrew formula push, all cut from
  one `vX.Y.Z` tag in lockstep.
- **SDK** (`python-sdk`, `typescript-sdk`, …) — bump the analyzer version
  pin to the just-released version, then the SDK's own release.

Never tag off a stale local build; the tag is what the pipeline trusts.

## Closeout

Docs updates, issue/epic bookkeeping, and following the propagation verdict
through — full etiquette in `references/docs-and-closeout.md`. In brief:
update the surfaces that describe what changed, close the child issue(s)
this work resolves (sub-issue progress rolls up on its own), and for every
repo the propagation verdict listed, file the follow-on issue before
stopping — a verdict that lists a repo and gets no issue is a dropped
thread, not a closed one.

## Terminal State

finishing-cldk-work is the ladder's exit. If the propagation verdict listed
follow-on repos, each becomes a new maintaining-cldk entry.

**Checkpoint before re-entering.** Report what shipped and what the propagation
verdict found, then `AskUserQuestion` — file the follow-on issues and start the
first one now, file them and stop, or stop here entirely (see
`using-cldk-devtools` → Transition Checkpoint). Re-entering the ladder is a
transition like any other; do not roll straight from a release into new work.
