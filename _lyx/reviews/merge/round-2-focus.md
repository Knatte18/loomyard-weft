---
round: 2
focus:
  - "Check `probeQuiet` in `internal/pairteardown/teardown.go` against `shedrun.DriverLock` and `shedrun.SelfRunID` in `internal/shedrun/paths.go`, which the parent's detached Go driver (`lyx loom drive`) holds for its whole life while leaving the run lock free between steps: does a live Go driver make `probeQuiet` report not quiet, so that `busyError` names it and `fabric remove` in `internal/fabricengine/remove.go` (`removeWarpWorktreeDir`) refuses rather than removing the worktree?"
  - "Check that the existing `probeQuiet` test in `internal/pairteardown` covers a held driver lock with a free run lock and a free loop lock, and that the driver-lock probe uses the same non-blocking try-acquire as `runLockFree`."
---

Focus targets the interaction between the parent's Go driver and the task's leftover-directory retry in `fabricengine`.
