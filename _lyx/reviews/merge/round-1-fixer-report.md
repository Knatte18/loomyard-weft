# Fixer-report, merge round 1

## Changes

- F1, fixed with a variation: `internal/battenshed/innerrun.go` gains failure-message constants for the re-step check (one per seam: driver strand, locks, build, dirty paths), the re-step marker and the Go verify-retry records.
  `restepSkipped`, `restepRecorded` (`innerrun_binaryrestep.go`) and `publishRetryDecision` (`publishretry.go`) now go through `seamFailed`, and each seam calls `seamRecovered` right after its own successful read.
  Variation from the `fix` key: the check has one episode per seam instead of one `failRestepCheck` keyed by `what`, so one seam recovering does not end another's episode, and `restepSkipped` no longer carries a `step` field since the message names the seam.
  Tests: `TestInnerRun_ReadFailuresLogOncePerEpisode` gains rows for the four re-step check seams and a subtest for the Go verify-retry read (`retryHarness` gains `goRetryErrs`).
  The marker read has no row: its scratch directory is fixed inside the test loop, so a read failure cannot be injected without new harness plumbing.
  Committed as `22aa94bfc`.
- F2, fixed as proposed: `binaryRestep` logs the dirty-path skip at Info when the list differs from `restepWatch.dirtyLogged` and at Debug otherwise; the list resets with a new stop key (whole `restepWatch`) and on a changed build.
  A subtest of `TestInnerRun_ReadFailuresLogOncePerEpisode` asserts one Info and Debug for the repeats.
  The line reuses `logAttempt`.

`lyx gate test ./internal/battenshed` is green.
The full `./...` and integration tiers were not run.

## Disputed

None.

## Deferred

None.
