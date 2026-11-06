# webster begin-batch reports a delete-order conflict as plan_order

`begin-batch` chose a `plan_conflict` cause from one fact: whether any batch was begun.
So every blocking finding after the first batch read `tree_moved`, including a `delete-before-reference` finding whose remedy is to reorder the plan.

## What changes

- `PlanConflictError` gains a third cause, `plan_order`, taken when every blocking finding is `delete-before-reference`, with or without begun batches.
- A `plan_order` refusal's way forward says to move each named delete after the card that still references it, in place of "match the named cards to the tree".
- A blocking set holding any other finding keeps `plan_defect` or `tree_moved`, since moving a delete would not settle it.
  An informational `delete-before-reference` does not count towards the set.
- The check name is now one constant, `planindex.CheckDeleteBeforeReference`, shared by `planglyph` and `websterengine`.
- The `begin-batch` help, the Master stencil, `websterengine`'s and `planglyph`'s package docs and `loom-plan-checks-spec.md` describe the cause and the check's reach.

## Why no check was added at plan acceptance or rebaseline

The issue asked to consider one, on the reading that the defect existed from the start.
The record of the `drift-rework` run shows otherwise, and neither check would have caught it:

- The plan gates already run `delete-before-reference`, typed, over the whole plan.
- The refusal named `internal/driftcli/wiring.go:247`, but at the run's start `compareLockRetry` was referenced only in `cli.go`, inside `compareCmd`, which the deleting card also removed.
  The reference at `wiring.go:247` was code the run wrote, so no plan gate could read it.
  The same holds for code inside a later card's Edit target that is still a `plan:` handle, which the check skips while it is unbound.
- Every `begin-batch` checks all pending cards, so a conflict is reported by the first `begin-batch` after the code lands.
  A plan edit accepted by `rebaseline` meets the same check at the next `begin-batch`, before any fork.

The bound is now stated in the checks spec and in both package docs.

## Left out

`uses-later-target` is also a plan-ordering finding and still takes `plan_defect` or `tree_moved` at dispatch.
It is outside this entry, and it reaches dispatch only through a mid-run plan edit, since it reads plan text alone and the plan gates refuse it.

## Tests

`TestBeginBatch_ConflictCauseNamesAPlanOrderConflict` covers the cause and way forward for an ordering-only set, mixed sets, and an informational finding beside a blocking one, each with and without begun batches where that matters.
