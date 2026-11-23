# Pin that a plan amendment keeps Webster's done batches

Issue #185 reported that `lym loom decision add --amends-plan` during Webster sent the run back to Plan-Burler-Review and lost Webster's `state.json`.
Plan-Burler-Fix then failed with `create-already-exists` on cards whose Creates had already landed.

The reported path does not lose the record.
No step of it (`decision add --amends-plan`, the goto to Plan-Burler-Review, Plan-Burler-Fix) moves `_lym/webster/state.json`.
Only Plan-Write's and Parts-Plan-Write's rotation, `webster run --fresh` and `webster reset --to start` do.
No run in this hub matches the report, so the incident is untraceable and the change adds a guard, not a fix.

Closes #185

## What changes

`internal/loomcli/amendduringwebster_test.go` adds `TestAmendsPlanDuringWebster_KeepsDoneBatches`.
It plants four done batches in Webster's run record, lands each card's Create in the worktree, and leaves a fifth card pending.
It then drives `decisionVerb` with `amendsPlan` and the engine's goto to Plan-Burler-Review.
It asserts two things:

- the record read through `websterengine.LoadState` after the goto equals the record read before it;
- the plan gate passes with no `create-already-exists` finding on the done cards.

The test passes on the merge base, so it is not a red-first test.
No production code changes.

## Design decisions

- The test reads the record through `websterengine.LoadState`, not through a filename literal, so it follows the record's location if that moves.
- The goto runs through `shedengine.Goto` with a four-producer chain, not loom's goto hooks, so loom's own hook behaviour is not covered.

## To check by hand

- `run-record-check-holds-by-construction` in `internal/loomcli/amendduringwebster_test.go`:
  the record assertion holds by construction while no step on the path writes the record, so it can only fail if a later change adds such a write.
  Decide whether that is the guard wanted.
- `webster-state-filename-literal` in `internal/loomcli/amendduringwebster_test.go`:
  the review's fix reads the record through `websterengine.LoadState` instead of a literal filename.
  Confirm the test names no `state.json` path itself.

## Merge danger

**Door:** two-way
**Blast radius:** loomcli
