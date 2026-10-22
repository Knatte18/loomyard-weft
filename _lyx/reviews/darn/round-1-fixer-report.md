# Round 1 fixer-report

## Changes

- F1, fixed as proposed, then refined after the self-check: the `archiveTagPushDelays` comment in `internal/fabricengine/archive.go` and the matching sentence in `_lyx/landing/summary.md` now say a teardown archives once and holds at most the pair's own teardown lock, which only that pair's detached pushes wait on.
  "At most" because `Cleanup` also calls `archiveWeftTip` and takes no teardown lock.
  Commits `fc748fd0b` and the follow-up refinement.
  The `summary.md` edit is uncommitted, since `_lyx` is owned by the loop.
- F2, fixed as proposed: `TestArchiveTagPushDelays` in `internal/fabricengine/archivetag_test.go` pins the schedule 2 s, 4 s, 8 s, 16 s, 32 s.
  Commit `dc84f6dc4`.
- F3, fixed as proposed: row "empty delays make one attempt" in `TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound`.
  Commit `eb9d51108`.

`lyx gate test ./internal/fabricengine ./internal/gitrepo` passed before the final comment-only edit.

## Self-check

- fix-code-correctness, LOW: the test pins the helper, not its use at the `PushWithRetry` call. Deferred, see below.
- fix-code-correctness, LOW: F3 row, no defect. No change.
- fix-consistency: no findings.
- fix-proposal, MEDIUM: "only the pair's own teardown lock" is false for `Cleanup`. Fixed with my own change: "holds at most".
- fix-proposal, LOW: same gap as the first line (call site unpinned). Deferred, see below.
- fix-proposal, LOW: F3 row, no breakage found. No change.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- Pinning that `archiveWeftTip` passes `archiveTagPushDelays()` to `PushWithRetry`: needs an injectable delay or sleep seam in the production function, beyond the review's proposed fix, which asked only for the helper assertion.
  Raised by two forks as LOW.
