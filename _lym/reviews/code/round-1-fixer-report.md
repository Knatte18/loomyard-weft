# Fixer-report, round 1

## Changes

- F1, fixed as proposed (`a115887b8`): `weftDirtyStop` in `internal/fabricengine/pull.go` and `upstreamAnchorStop` in `internal/fabricengine/anchorstop.go` name `lymVerbDir(...)` for their `lym fabric commit` and `lym fabric anchor --set` steps. The `upstreamAnchorStop` doc line for `codeWorktree` now says it is the root whose anchored directory the step runs in.
- F3, fixed as proposed (`30c5e8592`): `BranchRewritten` in `stopWayForward` returns one `lym fabric pull` line in `lymVerbDir(b.CodeWorktree)`.
- F5, fixed as proposed (`4b97a9946`): the `doc.go` sentence is narrowed to the failed-push notice and reconcile stop way forwards.
- F2 and F4, fixed as proposed in part (`9acf412c8`):
  - `export_test.go` gains `StopWayForwardForTest` and `WeftDirtyStopForTest`.
  - `failedpush_test.go` gains `anchoredHub`-style stop rows for the code-side `BranchRewritten` pull, both `anchorStopWayForward` variants, the records-side `dirtyWayForward` commit and `weftDirtyStop`.
  - It also gains an empty `CodeWorktree` records row and an invalid-marker (`../escape`) row for the `lymVerbDir` fallbacks.
  - Tier 0 of `./internal/fabricengine` passed with these.

## Disputed

- Fork finding (fix-proposal, LOW), possible regression if a `BranchRewritten` outcome leaves `CodeWorktree` unset. Ground: premise false. `originreconcile.go:717` sets `codeWorktree: worktree` for code targets and `:890` copies it to the outcome before `stopWayForward` runs; every outcome, including the moves path, is built there.

## Deferred

- F2, the anchored tests for `upstreamAnchorStop` (`anchorstop.go`) and the records conflict push step (`pull.go:442`): not done. The `pull_integration_test.go` diverged-weft fixture puts the warp (`.../002`) and the weft (`.../003/container/...`) in different directories, so `lymVerbDir` finds no marker and returns the root. My anchored edits failed in tier 1 for that reason and I reverted them. Covering these lines needs a hub-sibling pair fixture, which is a larger new fixture than this round's fix. Fork findings fix-code-correctness MEDIUM and fix-proposal LOW, same gap.
- Fork finding (fix-affected-set, LOW) `internal/adoptcli/adopt.go:231`: the adopt unmoved-branch line names the branch's root worktree for `lym fabric pull`. It needs `lymVerbDir`, which is unexported in `fabricengine`, so converting it needs a new exported API and a change in another package, outside the review's files. Worth raising with the parent.

## Rulings

None.

## Self-check

- fix-code-correctness, MEDIUM: F2 coverage gap for the `upstreamAnchorStop` and `pull.go:442` lines: deferred.
- fix-consistency: no findings.
- fix-proposal, LOW: possible unset `CodeWorktree` on `BranchRewritten`: disputed, premise false.
- fix-proposal, LOW: missing anchored coverage for `upstreamAnchorStop` and `pull.go:442`: duplicate of the F2 deferral above.
- fix-affected-set, LOW: `internal/adoptcli/adopt.go:231` still names the root: deferred.

## Gate findings

The verify run failed in `internal/gatecli` (`TestRedundancyStep`, "timed out after 1s") and in `internal/reedengine` (real-tmux tier 2, every subtest PASS, package FAIL after 285s).
Neither package is in this change set, and both read as timing failures under load.
Rerun alone, `lym gate test ./internal/reedengine --tier 2` exits 0.
The `gatecli` test failed again in the next verify run, under a machine load average of 24 to 45.
Its subtest `ABrokenLymBuildAndACorruptMapAreErrorLinesThatLeaveTheOtherTestsJudged` ran real `go` runs with a 50ms `secondUnit`, a 1s per-test timeout.
The repair raises that subtest's `secondUnit` to 500ms in `internal/gatecli/redundancy_integration_test.go`; no test in it hangs, so it costs no wait.
`lym gate test ./internal/gatecli --tier 1` then exits 0 (276 targets, 0 failed).
No production code changed for the gate findings.

A later verify run failed again, in another `gatecli` test (`TestGateTest_DetachedRun`, "timed out waiting for the stand-in's tier invocations") and in `reedengine` (every subtest PASS, package FAIL after 260s), at a load average near 50.
Neither package is in this change set.
Rerun alone, `lym gate test ./internal/gatecli --tier 1` and `lym gate test ./internal/reedengine --tier 2` both exit 0.
I made no change for this run; the failures are load-driven timeouts.

## Verification

Tier 0 of `./internal/fabricengine` passed. Tier 1 ran before the integration-test revert and failed only in the two `pull_integration_test.go` edits I then reverted; I did not rerun tier 1 on the final tree.
