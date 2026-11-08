# Drop the coverage study's removable fabricengine tier 1 tests

This change drops `internal/fabricengine` tier 1 tests listed in the coverage study's removable set, `greedy.tsv` in `docs/research/speed/fabric-test-redundancy-2026-11-08/`.
Those tests can go together without leaving a package-scoped block uncovered.
The study's list was the only authority: no other check vetoed a drop, and the module-wide `-redundancy` check was not run.
Production code does not change.

## Numbers

- In scope: 252 tests of the set.
  The set lists 329; the first cut (`f2ac2bf7`) had already dropped 6, 67 carried `//testtiming:keep`,
  and 4 more were skipped because the first cut folded them or moved their logic to tier 0:
  `TestWireJunctions_ScratchLink`, `TestRemove_UnlinksScratchLinkBeforeWorktreeRemoval`, `TestUnwire_LeavesSiblingWorktreeUndirtied` and `TestSourceDirtyPaths`.
- Dropped: 198. Their standalone cost in the study totals 51.5 s.
- Kept: 54, each marked `//testtiming:keep <reason>`; the reasons are listed below.
- Tier 1 time of `internal/fabricengine`, one `lym gate test --tier 1 ./internal/fabricengine` pass each, same machine:

| | Before the first edit | After the last commit |
| --- | --- | --- |
| Gate wall time | 41.4 s | 44.2 s |
| `go test` package time | 35.9 s | 34.2 s |
| Passed (tests and subtests) | 1677 | 1366 |
| Skipped | 173 | 173 |

The wall time did not fall.
The package time fell by 1.7 s, within run-to-run noise at this size.
The suite runs in parallel, and most dropped tests cost 0.2 to 0.5 s each standalone, so their time was spread across workers rather than on the critical path.

## What was kept, and why

A test was kept only when it asserts something concrete that block coverage cannot see:
archive tags and their order, deletes on origin, user content never claimed or removed, mutation records and their order,
or machine-local files kept out of the weft tree.
Commits and uncommitted edits the operator made count as user content.

- `TestRemove_ArchiveOrder`: the archive tag pushed after the weft branch on origin, and a diverged or ahead origin tip getting its own tag.
- `TestRemove_PendingRecordsAreCommittedAndArchivedOnce`: the post-commit tip carrying a pending uncommitted record getting the one archive tag on origin.
- `TestRemove_CommitsPendingRecordsIntoArchiveTag`: the pending record committed with a Warp-SHA trailer and carried into the archive tag, with one commit_created mutation.
- `TestRemove_FinishesPairWhoseTaskWorktreeWasRemovedByHand`: the pending record committed into the archive tag with a Warp-SHA trailer naming the task branch tip, after the task worktree was removed by hand.
- `TestRemove_FinishesPairWithBothWorktreesGone`: the archive tag on origin targeting the sibling branch tip before that branch is deleted, with both worktrees gone.
- `TestRemove_ArchivesAndDeletesSiblingBranchOnlyOnOrigin`: the archive tag targeting the origin-only sibling branch tip and that branch then deleted on origin.
- `TestAdd_DivergedArchivedWeftLeftoverReplaced`: the archive tag covering the old pushed weft tip, and the remote_branch_deleted mutation entry for the replaced origin weft branch.
- `TestRemove_RemoteTrueDeletesWeftBranchOnRemote`: the weft branch deleted on origin under remote, with exactly one remote_branch_deleted mutation record for it.
- `TestRemove_RemoteFalseLeavesRemoteBranchIntact`: the weft branch on origin left intact when remote is off.
- `TestRemove_RemoteDeletesSquashLandedTaskBranchOnOrigin`: the squash-landed task branch deleted on origin under remote.
- `TestRemove_WithoutRemoteLeavesTaskBranchOnOrigin`: the task branch on origin left untouched when remote is off, even once squash-landed.
- `TestCleanup_RemoteTrueDeletesLocalAndRemoteOrphan`: the orphan weft branch deleted on origin as well as locally under remote=true.
- `TestCleanup_RemoteFalseLeavesRemoteCopyIntact`: the origin copy of a deleted orphan left intact when remote is off.
- `TestCleanup_DryRunWithRemoteDeletesNeither`: a dry run with remote=true deleting the orphan branch neither locally nor on origin.
- `TestCleanup_ProtectedEntryUntouchedOnRemote`: a protected unmanaged branch left untouched on origin and locally under remote=true.
- `TestCleanup_NeverPushedOrphanIsIdempotentOnRemote`: no remote_branch_deleted mutation record for a branch that was never on origin.
- `TestCleanupRemoteWarp_BranchPrefixSetOverBareBranches`: the managed bare branch deleted on origin and the unmanaged one left on origin under branch_prefix.
- `TestCleanupRemoteWarp_BranchPrefixSetTakesTheBareSlugsLock`: the bare branch left on origin while the teardown lock of its own slug is held.
- `TestCleanupRemoteWarp_DryRunClassifiesEveryBranch`: a dry run recording no mutations and leaving every classified branch on origin.
- `TestAdd_LiveLocalWeftAdoptsDivergedWarpFromOrigin`: origin's diverged warp branch left untouched at its tip while the live pair adopts it.
- `TestAddRollback_AdoptedWarpBranchLocalCopyDeleted`: origin's warp branch left untouched at its tip when the rollback deletes the adopted local copy.
- `TestAdd_SkipPushSkipsLeftoverProbes`: both origin branches left unmoved and the leftover weft not deleted under SkipPush.
- `TestRemove_UnlandedWarpBranchIsKept`: a local task branch carrying unlanded commits surviving Remove, with and without force.
- `TestRemove_ReturnsPairNotFoundWhenNothingRemains`: a user's stray directory and its file surviving a Remove that finds no pair.
- `TestCleanup_ProtectsPrimaryWeftBranchAfterCheckout`: the primary weft branch and its commits surviving cleanup --apply --force after a checkout moved the prime pair.
- `TestCleanup_PrimaryBranchSurvivesForceWhenNotCheckedOut`: the weft primary's branch surviving apply+force Cleanup while no worktree has it checked out.
- `TestAddRollback_AdoptedWeftBranchSurvives`: an adopted weft branch surviving the rollback at its pre-existing unpushed commit.
- `TestPrune_ProtectsDirtyWeftWorktreeUntilForced`: uncommitted tracked weft work surviving prune --apply until --force.
- `TestPrune_RefusesHubDirectoryItDoesNotOwn`: a user's real `<slug>-weft` directory and its file surviving prune in every mode including --apply --force.
- `TestReconcile_PreservesUserSymlinkAtAnchor`: a hand-authored symlink at the anchor surviving the stale-junction sweep while a stale fabric junction is removed.
- `TestDotLymJunction_AdoptionDoesNotOverreachIntoLymOrPattern`: a user's real `_lym` or other junction-named directory and its file never adopted, claimed or changed.
- `TestMutateGitExclude_ConcurrentMutationsPreserveExistingContent`: the operator's pre-existing exclude pattern surviving concurrent mutations exactly once.
- `TestPull_DirtyWarpRefusesBeforeMovingWarp`: the user's uncommitted edit to a tracked warp file surviving a refused Pull byte for byte.
- `TestMerge_DirtyTargetHalts`: the operator's uncommitted weft edit left untouched by a warp-only Merge.
- `TestMergeIn_PairTurningDirtyWhileWaitingForLock_RefusesPreservingDirt`: the operator's uncommitted edit surviving a refused MergeIn instead of being reset away.
- `TestMergeAbort_ConcludeLandingWhileWaitingForLock_RefusesInsteadOfResetting`: a conclude landed while MergeAbort waited for the lock surviving, with no reset through the operator's resolutions.
- `TestMergeIn_StartsAreReReadUnderLock`: the recorded start SHAs on both sides naming a commit landed while MergeIn waited, so an abort cannot reset through it.
- `TestMergeCrucible_AbortRefusesAnAttemptWhoseConcludeLanded`: a conclude commit the operator landed by hand, unknown to the record, surviving a refused MergeAbort.
- `TestWeftGuards_AbortLeavesWeftCommitsDuringAttemptWindowIntact`: weft commits made during the attempt window surviving MergeAbort at the weft HEAD it found.
- `TestCoalescePushBothAt_DivergedWarpRemote_ReturnsNilWithoutSpinning`: the operator's warp working-tree file and `.git` left untouched, with no rebase attempted, when the warp push is rejected.
- `TestAdd_LiveLocalWeftAgainstOrigin`: the repo_advanced mutation entry naming origin's tip on a fast-forward, and an empty record with the local weft branch unmoved on a diverged refusal.
- `TestAdd_AdoptedWeftKeepsItsOriginRecord`: no file_written entry for the origin record and no commit_created entry on an adopted weft branch.
- `TestAdd_OriginRecordMutationEntries`: exactly one file_written entry for the origin record and one commit_created entry targeting the weft worktree with the landed sha.
- `TestAdd_SkipGitWritesRecordWithoutCommit`: a file_written entry for the origin record and no commit_created entry under SkipGit.
- `TestMerge_StaleTargetSyncsBeforeMerging`: the repo_advanced mutation record entry from the pre-merge sync fast-forward.
- `TestMergeIn_OneSideAlreadyUpToDate_OtherMerges`: no merge_staged or merge_committed mutation record entry targeting the already-up-to-date weft side.
- `TestMergeState_ResetMergeSides`: the single KindWorktreeReset mutation entry carrying the warp pre-merge SHA, and the weft's conflicted state left untouched.
- `TestResetPairCode_OursMergeThenRerunConverges`: the remote_branch_updated then worktree_reset records, in that order, and the remote task branch rewound after the ours merge.
- `TestReconcile_CorrectJunctionsAreNoOp`: a no-op Reconcile recording no link_removed mutation at the root `.scratch` link.
- `TestRemove_SweepsAnchoredLinksOnSubpathHub`: the scratch link_removed mutation recorded once, ahead of the warp worktree_removed.
- `TestMergeState_Record`: the merge-state record kept out of git status on both the warp and the weft.
- `TestCommitWeft_LockArtifactsExcludedFromStatus`: the `.weft` lock dir and legacy push lock being git-excluded from weft status while a stray lock under `_lym` still shows.
- `TestCloneHub_BoardStageAllCommitNeverStagesHubScratch`: a file planted in the hub scratch tree never appearing in the tree of the board's stage-all commit.
- `TestBolt_DirtyRepo_CommitsAndPushes`: machine-local scratch under `.lym` staying untracked by Bolt's wildcard commit while the pushed SHA lands on origin.

## Other test-code changes

- Helpers, layers and `export_test.go` shims that only dropped tests used are deleted, along with imports left unused.
- 14 files left with no test are deleted.
- `refuseRecordsCommits` moves into `mutation_record_integration_test.go`, its one remaining caller.
- File-header comments that described dropped tests are reworded to match what each file still holds.

## Design decisions

- The writer ran the tier 1 gate twice, only to take the before and after times; the parent recorded that in the decision record.
- Skipping the four tests the first cut folded was confirmed by the parent.

## To check by hand

- `docs/benchmarks/test-redundancy.md` and `docs/benchmarks/test-suite-timing.md` still name dropped tests.
  They are generated reports and are not rewritten here.
