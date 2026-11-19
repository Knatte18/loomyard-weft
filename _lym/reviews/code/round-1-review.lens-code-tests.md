```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that pins it.

**Per behaviour, the test that pins it**
1. D7: `MergeStateActive` reports a merge only, so a weft cherry-pick reports false. Pinned by `TestMergeStateActive/weft cherry-pick in progress reports false` (`internal/fabricengine/mergestateactive_integration_test.go`).
2. D7: a status commit during a records cherry-pick commits instead of skipping (acceptance criterion). Only the probe is pinned, by item 1. The seam-level row `TestCommitStatusSeam_Real/records cherry-pick in progress commits the transition` is deleted, and no seam-level row is left (finding F1).
3. D7: a deferred failed-push record that names commits sends no mail. Pinned by `TestFailedPushRecord/DeferredWritesTheRecordAndNeverMails`, which now sets `Commits` (`internal/fabricengine/failedpush_test.go`).
4. D5/D6: a protected refusal records the `other` class and is not retried. Pinned by `TestPushFailureClasses/AProtectedBranchRefusalRecordsTheRejectionClassAndIsNotRetried` (`internal/fabricengine/pushclasses_integration_test.go`).
5. D6: `ProtectedBranchRefusal` is read by landingshed's parent push. Pinned by the existing protected-parent rows of `TestFinalize_*`, which call `fabricengine.ProtectedBranchRefusal` through `pushParent`. No change at the call site except the package qualifier.
6. D6: gitrepo appends `remote:` lines to a rejection. Pinned only end to end by item 4's test: the hook text reaches `ProtectedBranchRefusal` only through those lines. No gitrepo unit row covers it (`internal/gitrepo/pushretry_test.go` has `remote:` noise, but only for `RetryablePushRefusal`). Acceptable as coverage.
7. D7: `Fabric.Commit` refusal order is restored to the pre-f06acf39 one. Pinned by `internal/fabricengine/mergesiblings_integration_test.go:103` (`*ErrMergeInProgress`) and `:290` (`*ErrForeignMergeState`). The mutation survivors at `commit.go:112/115` show that the mutation run's tier did not reach these tests. They are not a gap.
8. statuscommit without the wait, Stalled and Protected seams:
   - A push that is not owed halts the run: `TestNew_PushErrorReturnsNil` (rows `OwedReadsFalse`, `OwedNil`).
   - Skip on a merge: `TestNew_MergeActiveSkips`.
   - Re-probe skip: `TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip`.
   - Owed retry: `TestNew_OwedPushRetriesBehindTheGate`.
   - The board-status write before Run-Shed is unchanged and pinned by `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed`.
9. battencli and loomcli seams with no layout branch: the two-repository skip is pinned by `TestCommitStatusSeam_Real/mid-merge skips without touching the merge` (`internal/loomcli/wiring_commitstatus_integration_test.go:306`) and `.../merge going live after the probe skips instead of halting` (`:333`). For batten, the seam's pass-through of `MergeActive` is covered by `TestNewCommitStatusSeam`. The deleted `TestNewCommitStatusSeam_LayoutWiring` row `TwoRepositoryMergeInProgressSkips` was the only batten-seam row asserting a skip on `MergeActive` true. I could not confirm another batten row asserts it, so I report it as a candidate gap (F2).
10. runfsm: the push-owed refusal names `lym fabric push` in the owed worktree. Pinned by `TestAct/PushOwedWaitsFirst` and `PushOwedWithoutErrorOrWorktreeIsBare`.
11. pairteardown `ReadRun` fills no way forward. Pinned by the `TestReadRun` row with `wantIn` `{PushOwed, PushOwedError, PushOwedWorktree}`.
12. lymcwd: the anchor is read only from the records sibling, then Bolt; `ResolveWithin` has no layout case. Pinned by the `TestReadRecordedAnchor_MarkerSemantics` rows and `TestResolveWithin`. No row asserts that a stray root `.lym-anchor` in a code worktree is ignored. That is the pre-f06acf39 behaviour, so it is no new behaviour and no finding.
13. landingshed: the empty-landing flag goes through `Deps.ChangedPaths`. Pinned by `TestFinalize_RecordsWhetherTheLandedDiffIsEmpty` (rows `NoPathChanged`, `APathChanged`, `ThePathReadFails`, `NoMergedHeadHeld`). Only the stub's wiring is pinned; that `ChangedPaths` over a real two-repo pair excludes records is structural, since the code repository holds none.
14. Landing commit message has no `Fabric-Merge` trailer. Pinned by `TestFinalize_OverRealHub` (exact `wantMsg`).
15. loomcli `ownMergeInLeftover` and `refuseOverUnfinishedMerge` handle Parked only. Pinned by `TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals` and `..._CleanPairProceeds`. landingshed `preMerge.clear` and `parkedMergeWayForward` are pinned by `TestPreMerge_ClearsOwnParkedMergeIn` and `TestFinalize_StuckReasons`.
16. gitrepo `MergeStart` always passes `--ff`. Covered by the existing merge tests through fabric; no change for two-repo callers.
17. fabriccli clone has no `--single-repo`, and the engine's refusal reaches the envelope unchanged. Pinned by `TestCommand_CloneEngineRefusalReachesEnvelopeUnchanged`.
18. Acceptance criterion D5, "a records commit in the prime commits on any branch the prime is on": no row pins it. In a two-repository hub, which is the only layout left, the `lym.primeBranch` refusal never applied, so no two-repo behaviour changed. No finding.

**Findings**

- **F1.** MEDIUM, class scope.
  - Location: `internal/loomcli/wiring_commitstatus_integration_test.go`, the deleted row `TestCommitStatusSeam_Real/records cherry-pick in progress commits the transition` (diff 023, lines 217–245).
  - Summary: the only seam-level test of D7's acceptance criterion "a status commit during a cherry-pick in the records worktree commits instead of skipping" is deleted, with no `Loosened-Check:` trailer. It is two-repository coverage that c83fec49 added, and D5 licenses removing only one-repository tests.
  - Evidence: `.scratch/intervening.diff`, the c83fec49 hunk of `wiring_commitstatus_integration_test.go`, shows the row targets a two-repository hub ("A records cherry-pick in a two-repository hub does not count"). At HEAD, `TestCommitStatusSeam_Real` has no cherry-pick row (grep at lines 127–429).
  - Fix: restore the row and its helpers `startStoppedCherryPickInRecords` and `runGitAllowingFailure`.
- **F2.** LOW, class scope.
  - Location: `internal/battencli/commitstatus_test.go`, the deleted `TestNewCommitStatusSeam_LayoutWiring` (diff 004, lines 93–144).
  - Summary: the row `TwoRepositoryMergeInProgressSkips` pinned that batten's seam skips commit and push when `MergeActive` reads true. The whole test was deleted with its one-repository row.
  - Evidence: diff 004 lines 107–109. The remaining rows of `TestNewCommitStatusSeam` were not shown in my context. If `TestNewCommitStatusSeam` already has a merge-active row, drop this finding; otherwise keep the two-repository row.

**Tool calls**
- Grepped gitrepo tests for `remote:`, fabricengine tests for a prime switching branch, and the `t.Run` rows of `TestCommitStatusSeam_Real`, to locate the pins for items 2, 6 and 18, which my context lacked.
- Grepped gitrepo tests for `remoteLines`, `mapPlainPushError` and GH006, and read `pushretry_test.go` lines 15–40, to check for a gitrepo unit pin of the `remote:` lines (item 6).
