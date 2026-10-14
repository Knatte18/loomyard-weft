# Darn review budget overridable per run, with a separate off switch for the perch

`darn-run-budget` · Note · enhancement, config, shed

## Problem

Darn's `review_max_bounces` and `review_hard_max_bounces` are `CategoryGuard` keys (`internal/darnengine/config.go`), so `internal/configengine/layers.go` refuses them in a `.local` override and a board entry's `run_settings` cannot set them for one darn run.
The only way to change a darn run's review budget is hub-wide, in the hub's `darn.yaml`, which changes every darn run.
Loom's `review_max_bounces` is category none (minimum 1), so a loom run takes a per-run budget through `run_settings`, and loom has a separate guard key, `skip_merge_review`, for switching a perch off.

Darn guards the budget keys because `review_max_bounces: 0` skips Darn-Review and Describe: the budget key and the off switch are the same key.

Seen on 2026-10-14 with `tokencount-fork-output`: the operator wanted no review round on a small darn run, which no per-run setting allows, and `lyx loom goto` cannot move a run past a review row.

## Direction

The operator finds the difference wrong: darn and loom are both shed recipes and should read their config the same way.

- Darn's budget keys become ordinary keys like loom's, overridable per run through `run_settings`, with a minimum of 1.
- Switching the Darn review perch off becomes its own guard key, like loom's `skip_merge_review`, so the guard stays on the off switch and not on the budget.
- Check whether darn and loom can share one config shape for perch budgets and off switches, so the two cannot drift apart again.
