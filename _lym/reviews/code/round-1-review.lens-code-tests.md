```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per added or changed behaviour, the test that pins it, or none.

**Per-behaviour list**

| Behaviour | Pinning test | Verdict |
|---|---|---|
| `OneRepository` predicate, root marker wins, bolt and sibling excluded | `TestReadRecordedAnchor_MarkerSemantics` rows | pinned |
| `ResolveWithin` one-repo, suffixed checkout resolves as itself | `TestResolveWithin_OneRepository` | pinned |
| Marker-at-root read: empty marker gives ".", invalid marker is an error | `TestReadRecordedAnchor_MarkerSemantics` | pinned |
| Two-repository worktree with missing weft sibling reads as two (D1) | `TestRecordsGeometry_OneRepository` (`TwoRepositoryCodeWorktree`) | partial: that row has the sibling present in git, not missing |
| `RecordsPathsAt`, `IsRecordsOnlyCommit`, `IsFabricMergeCommit` | `TestRecordsPathsAt`, `TestCommitPredicates` | pinned |
| Records geometry (`RecordsWorktree`, `RecordsRepoRoot`, lock dir, `recordsBranchIn`) | `TestRecordsGeometry_OneRepository` | pinned |
| `Add` one-repo: real dirs, exclude, scratch link, no sibling, run-records dropped, trailer, adopt, rollback | `TestAdd_OneRepository` | pinned |
| Reconcile and `Healthy` report no junction cause, unwire no-op | `TestAdd_OneRepository` rows | pinned |
| Anchor change as one commit with trailer; stop on origin anchor | `TestAnchorChange_OneRepository` | pinned |
| `Add` run-records deletion carried in the first commit when the fork tracks them | `ATaskForkedFromAParentCarryingRunRecordsStartsWithNone` | pinned |
| Clone: `--single-repo` fresh origin, refusals (Bolt, binding, second URL), protected and refused marker push, one-URL over `lym-bolt`, missing-marker refusal, scratch not staged | `TestCloneAndWire_OneRepository`, `TestCommand_CloneEngineRefusalReachesEnvelopeUnchanged` | pinned |
| Marker push before `lym-bolt` push (order) | `ProtectedMarkerPushLeavesNothing` (no `lym-bolt` on origin) | pinned indirectly |
| `ensureBoardWorktree` tracking origin's branch, with upstream | `OneURLOverLymBolt` (upstream asserted) | pinned |
| Path-limited records commit, staged code kept, `Warp-SHA` trailer, empty commit with staged code | `TestRecordsCommit_OneRepository` | pinned |
| `index.lock` retry | `TestRetryHeldIndexLock` | pinned for the helper; no row shows a real commit path using it |
| Prime off branch or key absent refuses; two-repo prime not refused | `TestRecordsCommit_OneRepository` | pinned |
| Records commit during a merge: transition, abort variants, sync, code-path refusal, cherry-pick, crash before step 3, continue refusal, unsettled abort keeps an edit | `TestRecordsCommit_DuringMerge` | pinned |
| `MergeStateOperation` / `MergeStateActive` | `TestMergeStateActive` one-repo rows | pinned |
| Merge strips records (merge, squash, merge-in), fast-forward source makes a merge commit, anchor-change merge, marker-edit refusal, revert OK, write between steps refuses, parked merge refuses | `TestMerge_OneRepository` | pinned |
| `MergeContinue` records-unsettled refusal; `MergeAbort` commits records first | `TestRecordsCommit_DuringMerge` | pinned |
| `Merge` pre-lock guard order: already-in-progress checked before step 1 commits records | `MergeOverAParkedMergeRefusesAndKeepsItsResolution` (`MergeIn` only) | partial: `Merge`, `squash` and the dirtiness guards are not covered over a parked merge |
| `syncSideBeforeMerge` one-repo replay and `FastForwardPair` levelling | `TestReplay_OneRepository` rows | pinned |
| `MidMergeRecordsUnsettled` consumers (landingshed, loomcli, `premerge`) | `TestFinalize_StuckReasons`, `TestPreMerge_ClearsOwnParkedMergeIn`, `TestRunDriverSpawnAndWait_MidMerge_*` | pinned |
| `MergeGuardError.Parked` text | `TestMergeErrors_PinnedStrings` | pinned |
| Merge trailer on every conclude (two-repo included) | `TestFinalize_OverRealHub` commit message | pinned for the landing; no test of a plain `lym fabric merge --continue` message in two-repo |
| Pair push once, refused push one owed record, takeover keeps transitions, no owed code tip | `TestPairPush_OneRepository` | pinned |
| `statusPushRequest` one-repo request shape (`WarpPath` empty) | spawn path | none directly |
| Teardown: archive tag, delete branch, delete view ref, ref already gone | `TestRemove_OneRepository`, `TestRunTeardownChild_...OneRepository...` | pinned |
| Older teardown job with no `DeleteViewRef` | none | none |
| `deleteOneRepositoryBranchOnRemote` lease rejected, kept | none | none |
| View ref delete failing, way forward text | none | none |
| Replay: records-only, `Warp-SHA` rewrite, origin code commit, ended runs set aside, local code stop, merge-trailer code commit not replayed | `TestReplay_OneRepository` | pinned |
| `Warp-SHA` naming a non-replayed commit stays | none: the rows only use trailers that name replayed commits | none |
| `rewordWarpSHA` failure restores the starting tip | none | none |
| Origin reconcile reports `BranchDiverged` on a code stop (`originreconcilemoves.go`) | none: no origin-reconcile one-repo test | none |
| `settleUnmovedWeft` and `weftCodeCommitsStop` | `LocalCodeCommitRewritesNothingAndStopsNamingIt` (Pull) | pinned |
| `integrateweft` code-commit stop (anchor-change integration) | none | none |
| Prime guard: operator commit, detached push, fast-forwarded commit, merge trailer on code, records-only and fabric merge push, protected, takeover entry | `TestPrimePushGuard` | pinned |
| `retryWeftPush` replayed-retry guard path | none directly | none |
| `ProtectedBranchRefusal` / `IsPushProtected`, only for a one-repo prime | `TestPushFailureClasses`, `TestIsPushOwed` | pinned |
| `operator_unpushed` read as owed; `FailedPushWayForward` for the new classes | `TestFailedPushRecord`, `TestIsPushOwed` | pinned |
| `PushOwedWayForward` in `runfsm` and `pairteardown.ReadRun` | `TestAct`, `TestReadRun` | pinned |
| Deferred record mails only with commits | `DeferredWithCommitsMailsOnceNamingThemAndTheMerge` | pinned |
| `statuscommit` wait, probe error, stalled commit failure, protected push completes | `TestNew_MergeOperationWaits`, `TestNew_PushErrorReturnsNil` | pinned |
| `statuscommit` real wait over a cherry-pick and a records-unlike-HEAD merge | `TestCommitStatusSeam_Real` | pinned |
| `battencli` seam wiring (`OneRepository`, `Stalled`, `Protected`) | none: `battencli` has no one-repo seam test | none |
| `ErrMarkerChange` wording | none in `mergeerrors_test.go`, though `SourceMarkerEditRefuses` checks the error type | partial |
| `hubforge.NewOneRepoHub`, `SeedConfig`, `CloneSecond` | `TestNewOneRepoHub` | pinned |
| Board write on `lym-bolt` | `TestBoardWrite_OneRepositoryHub` | pinned |
| `gitrepo.MergeStartNoFastForward` and `remoteLines` | covered only through fabric tests | no `gitrepo` unit test |

**Findings**

1. **MEDIUM, class design.** `battencli`'s one-repository seam wiring has no test.
   - Location: `internal/battencli/commitstatus.go:173-183`, `:233-249`.
   - `battenCommitStatusDeps` now fills `OneRepository` from `lymcwd.OneRepository`, `MergeOperation`, `Stalled` and `Protected`. `newCommitStatusSeam` swaps `MergeActive` for `MergeOperation` when `OneRepository` is set.
   - Only `loomcli` has a one-repository seam row. A broken branch swap in `battencli` fails nothing.
   - The diff's only `battencli` test change is a log-string tweak in `overview_test.go`.

2. **MEDIUM, class design.** The D5 ordering rule is pinned for `MergeIn` only.
   - Location: `merge.go` `Merge`, `mergeIn` and `syncSideBeforeMerge`; the row at `merge_onerepository_integration_test.go:512`.
   - The rule is that a merge verb run over a parked merge checks the already-in-progress guard before step 1 and before the dirtiness guards commit records, so neither aborts the merge first.
   - `Merge` (with and without `Squash`) over a parked merge has no row. `Merge` reaches `commitReceivingRecords` through `syncSideBeforeMerge` and `prepareOneRepositoryMerge`, so it is the verb with a separate path.

3. **MEDIUM, class design.** The D7 rule that a `Warp-SHA` trailer naming a non-replayed commit stays has no row.
   - Location: `replay.go` `rewordWarpSHA`; `TestReplay_OneRepository`.
   - Every row's trailer names a replayed or fork-point commit, so a rewrite of every trailer would still pass.
   - Add a row with a commit whose `Warp-SHA` names an unreplayed commit and assert the trailer is unchanged.

4. **MEDIUM, class design.** `originreconcile` has no one-repository test.
   - Location: `originreconcilemoves.go:277-283`.
   - The `localCodeCommitsStop` case, which maps to `BranchDiverged` with the commits named, is untested. D7 lists origin reconcile as a replay site.

5. **LOW, class design.** `integrateweft`'s one-repository stop has no row.
   - Location: `integrateweft.go:603-610`.
   - Only `TestAnchorChange_OneRepository` exists, and it runs a clean move. The stop on a local code commit, and the records-only carry with paths moved, are unexercised.

6. **LOW, class design.** The one-repository teardown error branches are untested.
   - Location: `remoteteardown.go`: `deleteOneRepositoryBranchOnRemote` (lease-rejected kept, gate refusal) and `viewRefError` with its `failedRemoteSteps` way forward.
   - The older-job-without-`DeleteViewRef` default is untested as well.

7. **LOW, class design.** D1's "weft sibling missing still reads as two" case has no row.
   - Location: `TestReadRecordedAnchor_MarkerSemantics`; `TestRecordsGeometry_OneRepository` (`TwoRepositoryCodeWorktree`).
   - The `lymcwd` rows put a marker at the root, which is the one-repository case. No row has a two-repository worktree with no marker at its root and no sibling on disk.
   - The geometry test covers the case in fabric with the sibling present. The acceptance criterion names the missing-sibling case.

8. **LOW, class design.** `statusPushRequest`'s one-repository branch has no direct row.
   - Location: `spawn.go:165-168`.
   - Detached pushes are covered only for the prime (`CoalescePushBothAt` with the same path twice). A task pair's detached push through `SpawnStatusPush` is not exercised.

9. **LOW, class design.** `gitrepo`'s new `MergeStartNoFastForward` and the `remoteLines` rejection-detail merge have no `gitrepo` test.
   - Location: `internal/gitrepo/merge.go`, `push.go:264-290`.
   - Fabric tests reach them. The `remote:` lines are asserted in the clone refusal text only.

10. **LOW, class design.** `retryHeldIndexLock` has no real-path row.
    - Location: `commitweftpaths.go`, `recordsgit.go`.
    - Only the helper has a row. A commit that meets a real held `index.lock` and lands within the bound (D4 acceptance) has none. If the wrapper in `commitRecordsPathsHeld` were dropped, no test would fail.

11. **LOW, class design.** The merge trailer on a plain two-repository `lym fabric merge --continue` has no row.
    - Location: `mergelifecycle.go` `mergeCommitMessage`.
    - The comment says every fabric merge in either layout carries `Fabric-Merge`. Only the landing commit message is asserted.

Tool calls: none. My context already held every file I judged.
