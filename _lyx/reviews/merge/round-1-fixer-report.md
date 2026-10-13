# Fixer report — merge round 1

## Changes

- F1 (fixed as proposed): `probeQuiet` in `internal/pairteardown/teardown.go` now also probes `shedrun.DriverLock(task, shedrun.SelfRunID)` with `runLockFree` and reports not quiet while it is held.
  `quietState` gained `goDriver`, and `busyError` names "the Go driver is live", so `fabric remove` refuses with `ErrDriverBusy`.
- Added `TestProbeQuietSeesTheGoDriverLock` in `internal/pairteardown/probequiet_integration_test.go`, tagged `integration` because an untagged test may not spawn git (`PATTERN-test-speed`).
  It covers a held driver lock with the run and loop locks free.
- Raised `internal/pairteardown` to 7 in `cmd/lyx/testdata/test-budget.yaml`.
- The optional rewording of the premise comment in `internal/fabricengine/remove.go` was not made: the comment on `removeWorktreeLeftover` in `internal/fabricengine/destroy.go` already claims only a late writer's error and holds once the driver is quiet.

Gates: `lyx gate test` and `lyx gate test --tags integration` pass for `./internal/pairteardown` and `./cmd/lyx`.

## Disputed

None.

## Deferred

None.
