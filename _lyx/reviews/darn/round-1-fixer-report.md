# Fixer-report, round 1

## Changes

All four findings are fixed as proposed.
`lyx gate test ./internal/orchengine` is green.

- F1 (MEDIUM): `TestWatcher_HandoffTimeoutHoldsWhileTheReadLookupFails` in `internal/orchengine/watcher_test.go` fails `mailreadat` once on a phase 200s past the post.
  It asserts the phase stays handoff-requested with `Stuck` set to the failure and `StuckByCreel` true.
  A second tick, with the lookup answering, asserts the phase times out from the read.
- F2 (LOW): `TestWatcher_HandoffTimeoutDiscountsAHubPauseAfterTheRead` in `internal/orchengine/watcher_test.go` opens a 50s hub pause after the read and asserts the phase holds at 60s unpaused and times out at 100s unpaused.
- F3 (LOW): no new test function.
  - `TestResetForFreshLaunch` in `internal/orchengine/state_test.go` seeds a non-zero `RequestReadAt` in both subtests and asserts it is zero.
  - `TestWatcher_HandoffTimeoutCountsFromTheRequestRead` seeds a stale `RequestReadAt` before the cycle start, which the start must clear.
  - `TestWatcher_HandoffCompleteClearsThenResumes` asserts the read is cleared on entering clearing, which covers `enter`.
- F4 (LOW): the sentence at `internal/orchengine/doc.go:190` now says the phase does not time out while its note request has no creel read, a request not yet posted included.
  The code is unchanged.

Commits: F4 and the F1, F2, F3 tests, in two commits because the three test changes share two files.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
