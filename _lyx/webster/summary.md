# beam-start-and-ownership: stopped at batch 07 on a plan conflict

Four batches are done and committed, and the run is stuck before batch 07.

## Done

- **01-remove-beam-promote** (`30f5bf87`): removed `lyx beam promote`. Deviations: `internal/beamcli/create.go`, `internal/boardengine/ownerset_integration_test.go`, `internal/fabricengine/destroy.go`. No departures.
- **02-end-processes-by-role** (head `8ef5bf29`): `runproc.EndRole`, and owners on root and beam projects only (the batch holds cards 2 and 3). Deviations: `internal/boardengine/owners.go`, `internal/boardengine/ownerset_integration_test.go`, `internal/boardengine/render.go`, `internal/runproc/end.go`. No departures.
- **04-adopt-projects** (head `5160c1d5`): `lyx adopt` takes beam projects and `root`, and `RunHolds` is added (the batch holds cards 4 and 5). Deviations: `internal/adoptcli/adopt.go`, `internal/adoptcli/adopt_integration_test.go`, `internal/adoptcli/adopt_test.go`, `internal/boardengine/export_test.go`, `internal/boardengine/project_test.go`, `internal/boardengine/runlock_test.go`, `internal/loomcli/peers_test.go`. No departures.
- **06-recipe-named-pawl** (`e3b50d7e`): the `beam` recipe is now `pawl`, and a `beam` seed reads as `pawl`. Deviations: `internal/loomrecipe/pawlrecipe_test.go`, `internal/loomrecipe/recipe_test.go`, `internal/shedcli/table_test.go`, `internal/shedrun/seed.go`. No departures.

The digests listed no renames of planned symbols. Batch 06's warnings were informational rename candidates against card 16.

## Stuck

`begin-batch 07` refused with `plan_conflict: tree_moved`. Card 16 (`pawl-recipe-rows`) still names `internal/loomrecipe#TestBeamRouting` and `internal/loomrecipe#TestNewBeam`, which batch 06's rename to `pawl` removed from the tree.
The way forward is a plan edit of card 16, then `lyx webster rebaseline --card 16`, then `lyx webster run` to resume at batch 07.

Batches 07, 09, 10, 11, 15, 17, 21, 25 and 26 are not begun.
