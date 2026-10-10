# Fixer report — plan, round 2

I checked both findings against the plan and the code.
Both premises held, so both are fixed.
`lyx loom validate-plan` passes after the fixes.
Its one informational finding, `premerge_test.go` as a new unit, was already there before this round.

## Fixed

- **F1** (the abort-failure Stuck reason has no test): confirmed.
  Card 5's planned `premerge_test.go` rows never made `AbortMerge` fail.
  The card now plans a row where the stop succeeds and then `AbortMerge` fails.
  The row asserts a Stuck reason naming the parent branch, `lyx fabric merge --continue` and `lyx fabric merge --abort`.
  It also asserts no path list and no merge-in, meaning the fake resolver is not called.
  No target changed: `premerge_test.go` is already a `Create` target.
- **F2** (the parent-branch read error has no test row): confirmed.
  `TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses` is at `internal/loomcli/start_midmerge_test.go:209`.
  Card 6 now extends it with one row: a parked `merge-in` state where the `recordedParentBranch` seam returns an error.
  The row asserts a plain error that carries the seam's error, that nothing is put to work, and that the bootstrap lock is released.
  `start_midmerge_test.go` is already an `Edit` target on card 6.

## Disputed

None.

## Deferred

None.
