# Fixer-report — webster round 1

## Changes

- **F1** (fixed as proposed, commit `16464c2e2`):
  `run.boundaryHeld` is now cleared in the gated-Done arrival branch before the covered check (`internal/shuttleengine/wait.go`), after the pending-send and re-prompt sends that clear `gateAtBoundary` (`wait.go`), and in `boundaryIdle` (`internal/shuttleengine/send.go`).
  Regression row in `TestGate_HoldsItsBoundaryWhileCovered`: a gated Done arrives on the first uncovered tick after a covered one, and the pending gate closure runs once inside `gateRecheckEvery`.
  The row fails with the arrival-branch clear removed (2 runs, want 1) and passes with it.
- **F2** (fixed as proposed, commit `378c1d879`):
  In `configurePlatformGroupTerminate`, a SIGTERM failing with ESRCH still returns the error, and any other failure returns `cmd.Process.Kill()`; neither arms a timer.
  The `ConfigureGroupTerminate` doc in `procgroup.go` and the Linux function doc now say so.
- **F3** (fixed, same commit):
  New `internal/proc/procgroup_linux_test.go` with rows for a cancel that never ran (`awaitGone` returns at once, no signal reported) and a cancel after the group is gone (ESRCH returned, one SIGTERM reported, `awaitGone` returns at once).
  The non-ESRCH kill row F3 asked for is deferred; see below.
- **F4** (fixed as the review's second option, commit `7860403a0`):
  The overview cuts beyond card 8 stay; an empty commit records them as intended and names the three backgrounds that state each rule.

## Disputed

None.

## Deferred

- The F3 row for a SIGTERM that fails with an error other than ESRCH, such as EPERM, and the shell being killed afterwards.
  The error cannot be produced against a group the test's own process started, and a seam that injects the signal call would add production surface and a package variable that blocks parallel tests, which is outside this finding's scope.
  The branch stays unpinned by a test.

## Self-check

- fix-code-correctness, LOW: the non-ESRCH kill branch has no test — deferred, as above.
- fix-proposal, LOW: the same untested branch — deferred, as above.
- fix-consistency: no findings.
- fix-affected-set: no findings.
