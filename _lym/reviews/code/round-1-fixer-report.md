# Round 1 fixer-report

## Changes

- F1 (MEDIUM): fixed with my own change, commit `c9c86ee8`.
  Added `TestPull_UpstreamAnchorStopRunsInTheAnchoredDir` in `internal/fabricengine/pull_integration_test.go`.
  It builds a hub anchored at `backend` with a `task` pair, gives the pair's weft an upstream, and has a second clone commit a different anchor marker on origin.
  `Pull` stops with `ErrAnchorChange`, and the test asserts the way forward names ``run `lym fabric anchor task --set other` in <warpPath>/backend``.
  A fall back to the worktree root in `lymVerbDir(f.warpPath)` makes the assertion fail.
  The proposed table row was not used: its fixture (`buildReconcileFixture`) makes `warpPath` a plain warp repo, not a hub pair.
  There `lymVerbDir` takes its root fallback whatever the anchor, so such a row could not pin the anchored directory.
  A hub-pair fixture was needed instead.
  The tier-1 gate over `./internal/fabricengine` passed with the new test run.
- Gate finding: the new test's doc comment had a line of 120 characters or more.
  A first repair split the sentence mid-phrase, which the gate flagged as a wrapped break.
  The first sentence is now shortened to fit one line: "covers a pull stopped by origin's changed anchor marker".

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

All four forks returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
