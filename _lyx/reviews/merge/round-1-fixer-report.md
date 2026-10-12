# Fixer report — merge, round 1

## Changes

- F1: the alignment attempt timeout now counts awake, unpaused time.
  - `internal/alignment/session.go`: `SessionConfig` gains `Pause` (a `pauserecord.Geometry`) and `Awake`; the `Session` holds a `pauserecord.SleepWatch` fed on `Begin` and every `Poll`.
    `Begin` resets `SpawnedAt` and `Deadline` after `Shuttle.Start` returns, so a start held by a pause spends nothing.
    `Poll` times out when `pauserecord.AwakeSince(SpawnedAt, now, sleeps, coverage)` reaches `Deadline - SpawnedAt`, and the probe spec carries the remaining awake time.
  - Wiring: `newParentReviewAligner` reads `hubgeom.PauseGeometry`; `landingAlignmentDeps` takes the geometry as a parameter, and `landingDeps` passes `runner.PauseGeometry()`.
    It is a parameter, not a call on the runner inside, because `TestLandingAlignmentDeps_Config` passes a nil runner.
  - Tests: a new subtest in `TestSession` covers a Poll after a paused interval and after a detected sleep (pending), then a timeout once the awake unpaused time is spent.
    The harness awake clock follows the fake wall clock, so the existing deadline test sees no sleep.
    `pauserecord` is added to the import allowlist in `seam_enforcement_test.go`; it takes told paths and resolves no geometry.
- F2: `internal/landingshed/pralign.go` no longer reads `Lock.PauseRequested` in `wait`; the `pausedReason` constant and its test case in `pralign_test.go` are removed.
  `lyx loom pause --after PR-Align` now halts at the row's boundary, and `lyx pause <slug>` is the mid-row stop.

`lyx gate test ./internal/alignment ./internal/landingshed ./internal/loomcli` is green.

## Disputed

None.

## Deferred

None.
