# Plan-review round is dispatched as a fix round after the plan is already implemented, and webster refuses the resulting plan edits

`plan-review-after-implemented` · Note · bug, loom, burler, webster

Imported from [issue #465](https://github.com/Knatte18/loomyard/issues/465).

A Plan-Review round ran over a plan that was already approved and implemented by webster.
The review gained findings, but the fixer's only write surface was the plan, and webster could not accept a new card on this run's record.
`lyx webster rebaseline --card=[11,12]` was refused:

```
webster: the edited plan changes the cards of a batch this run already begun: 00-overview.md changed and this run recorded no overview frame, so the change cannot be confined to its Card Index; it carries the plan's integration verify and is never rebaselined; way forward: restore 00-overview.md, or 1) lyx webster reset --to start; 2) lyx webster run
```

The parent stopped all fixes mid-round, so every finding ended deferred.
The fixer also spent its orientation designing a new card that was then thrown away.

Why this is abnormal: the loom dispatches a review round whose fixes cannot land.
A frozen plan (webster already begun) makes the fix round unwinnable, and the only advertised way forward (`lyx webster reset --to start`) discards the implemented work.
The run records no overview frame, so a legitimate edit to the overview's Card Index cannot be confined and is refused outright.

What to do:
- Skip the fix-round dispatch, or tell the round prompt up front that the plan is frozen, when the run's webster record has begun a batch.
- Record an overview frame when webster begins a batch, so a Card Index change can be confined and rebaselined.

Provenance: task `shuttle-send`; notes `burler-plan-r1.md`, `webster-refusal-rebaseline.md`.
