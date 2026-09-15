---
name: using-cldk-devtools
description: Use when starting any session or task that touches a codellm-devkit repository — before any action, including quick fixes, questions, and issue triage.
---

## The Rule

Before ANY action on a codellm-devkit repo — including answering questions
and "quick fixes" — find your entry point in the routing table. If a ladder
skill applies, you do not have a choice about *whether* the ladder governs
the work — but you do not enter or move along it silently. See Transition
Checkpoint.

## Transition Checkpoint

**Never invoke the next skill without asking first.** Every movement on the
ladder stops for the user: entering it, moving to the next rung, escalating
through a gate, and exiting it.

At each transition:

1. **Say where you are** and what you just finished.
2. **Name the next skill and why it is next.**
3. **`AskUserQuestion`** — proceed now, take a different path, or stop here.
4. **Invoke only after the answer.**

This governs the *route*, not the *gates*. A gate's verdict is a fact about the
work ("this moves schema v2, therefore it is structural"), and the checkpoint
does not reopen it — you still report the verdict plainly. What the user decides
is whether to go there **now**, go somewhere else, or stop. Never present
"skip the gate and patch it anyway" as an option.

**Stopping is always a valid answer.** A run that ends at a triage verdict, or
parks after the spec is written, is finished work — not abandoned work. Batch
nothing across a checkpoint: do not start the next rung's work "to save time"
while asking.

## The Ladder

```
                     using-cldk-devtools  (dispatcher)
                              │
   too plural for one design  │
              ▼               │
   planning-cldk-work         │   roadmap doc + ONE parent issue
        │ pick one decision   │
        ▼                     ▼
        structural work       │ upkeep work
              ▼               ▼
   designing-cldk-changes   maintaining-cldk
        │ datamodel → spec + record │  HARD GATE: escalate to design mode
        ▼                           │  if the fix moves schema v2 / public API
   codeanalyzer-backend             │
        ▼                           │
   cldk-sdk-frontend                │
        ▼                           ▼
           finishing-cldk-work  (verify → release → docs → close issues)
                              │
                    (future rung: cocoa)
```

## Routing

| Work type | Entry point | Path |
| --- | --- | --- |
| **Work too plural for one design session** — a theme that decomposes into several contract decisions ("microservice analysis"), or several initiatives competing for a quarter | planning-cldk-work | plan → design (one decision) → … |
| New language for CLDK | designing-cldk-changes | design → backend → frontend → finishing |
| Schema v2 evolution / migration | designing-cldk-changes | design → backend (all affected analyzers) → frontend (all affected SDKs) → finishing |
| New analysis level (L2/L3/L4) for a language | designing-cldk-changes | design → backend → frontend (if surface changes) → finishing |
| New facade surface / SDK feature | designing-cldk-changes | design → frontend → finishing |
| Bug fix (analyzer or SDK), behavior-preserving | maintaining-cldk | maintain → finishing |
| Small feature, no contract impact | maintaining-cldk | maintain → finishing |
| Docs gap / README / agent-guide update | maintaining-cldk | maintain → finishing (docs path) |
| Issue triage ("is this real?") | maintaining-cldk | maintain (may stop at triage verdict) |

## Red Flags

| Rationalization | Reality |
| --- | --- |
| "It's just a small schema tweak" | Schema changes enter at designing-cldk-changes. |
| "They named two things, so this is planning" | Count contract decisions, not nouns. The test is: *can this be stated as ONE contract decision?* Yes → design mode. Go L3/L4 is two levels of one decision, not a roadmap. |
| "They named one thing, so this is design" | Same test, other direction. "Microservice analysis" is one *theme* containing several decisions — boundary edges, topology, entrypoints — so it is planning even though they named one thing. |
| "We're planning several things, I'll design them all now" | Planning is shallow across many; design is deep on one. Run the collision sweep, then design the one that starts. |
| "I'll patch the SDK model directly" | Check the schema contract first — enter the ladder. |
| "This fix is analyzer-local" | Siblings share the schema. maintaining-cldk runs the propagation sweep. |
| "I'll release manually just this once" | Releases go through finishing-cldk-work. |
| "The next rung is obvious, I'll just invoke it" | Obvious to you is not agreed with the user. Checkpoint every transition, including the obvious ones. |
| "I'll ask at the end, once I've made progress" | The checkpoint is before the transition, not after. Work done past an unasked transition is work the user never approved. |

## Scope Guard

If the task does not touch a codellm-devkit repository, this plugin stays
silent. Do not route unrelated work through this ladder.
