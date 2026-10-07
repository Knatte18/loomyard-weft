# Round 1 fixer report

All three findings fixed; none disputed, none deferred.
Each fix is its own commit, and `go test` passes for `burlerengine`, `burlercli`, `shedadapters`, `loomcli`, `contracts` and `cmd/lyx`.

## Fixed

- **F1** (MEDIUM, design): `join` in `internal/burlerengine/handoff.go` now also stops a half whose result is `OutcomeTimeout`, through `stopHalf`, before it decides the round; a failed stop returns `ErrHalfNotStopped` as for any other half.
  The `join` comment and the "How the round ends" section of `internal/burlerengine/doc.go` say so.
  In `engine_test.go`, the fixer-timeout row of `TestEngine_Run_RoundFailureRules` now expects both guids removed, and a reviewer-timeout row was added to the same table; no new test function.
- **F2** (NIT, consistency): the three "No model/effort key" comments in `contracts/recipes/loom-recipe.yaml` lost their trailing repeated clause.
- **F3** (NIT, consistency): the `Long` help paragraph in `internal/burlercli/cli.go` is re-wrapped to the width of its neighbours.

## Deferred

None.
