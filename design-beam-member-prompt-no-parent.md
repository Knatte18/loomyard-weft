# Tackler and driver prompts of a beam member say no parent is recorded

`beam-member-prompt-no-parent` · Note · bug, stencil, shed

Imported from [issue #231](https://github.com/Knatte18/loomyard/issues/231).

## Seen

In the DrillOpPlan hub on 2026-11-10, the shed tackler of run `models-backend`, a member of the beam `adopt-models-sim`, got a prompt whose section "Your parent" read:

```
No parent is recorded for this run.
When you are blocked, write your report and stop.
```

The run had a parent: the beam's orch `dop:adopt-models-sim:orch`, which owned and released it.
Other tackler and driver prompts in the hub carried the same text, in the form "No parent is recorded for the run, so the question is written for whoever owns it."
The tackler escalated on a merge conflict and stopped; the prime orch saw the stop and passed it to the beam orch by hand.

## Expected

The tackler and driver prompts of a beam member name the beam's orch as the parent, so a blocked tackler or driver reaches it directly.

## Workaround

The prime orch forwards a beam member's stop to the beam's orch by creel.
