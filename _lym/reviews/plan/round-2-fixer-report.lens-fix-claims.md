```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: check that every claim the fix added or rewrote holds against the code at HEAD.

no findings

Claims checked, all of which hold:
- **`00-overview.md`, deletion estimates.**
  - Card 15's further deletions come to about 700 lines: the whole tests add up to about 280, the subtests to about 333 and the tails to about 90.
  - Cards 2, 3 and 4 plus `TestList_RefStorage` come to about 520, against the stated "about 530":
    - the four removed files are 211 lines;
    - the partition test and its helper about 55;
    - `TestBoardDir`, `TestHubPath`, `TestOriginRecordPaths`, `TestHubReservedNames_Still…` and `TestRecordsBranchName_RoundTrips…` 142;
    - the subtest and loop cuts about 55;
    - `TestList_RefStorage` 54.
  - The net of "about 180 lines down" follows from those figures. The 1,050 added lines is an estimate of code not yet written, so it cannot be checked against HEAD.
- **Card 6, the new bullet.** The record-value `Extend` calls are exactly the three it names:
  - `originreconcile.go:815`, in `ReconcileOrigin`;
  - `originreconcileended.go:125`, in `reconcileEnded`;
  - `teardown.go:584`, in `applyTo`.
  - Every other `Extend` call is on a recorder (`weft_verbs.go:286`, `clone.go:68`, `destroy.go:2255`).
  - `TeardownReport` embeds `MutationRecord`, so `report.Mutations` is the record value.
  - `s.rec` is a `fabricengine.Mutations` field of a value receiver, so the pointer method `Snapshot` can be called on it.
  - `applyTo`'s doc says the snapshot's record goes ahead of the removal's own, which matches the bullet's order.
- **Card 15, holder tests.** Every named holder test and subtest exists in `internal/fabricengine`:
  - `TestRemove_CommitsPendingRecordsIntoArchiveTag`
  - `TestMergeSiblings_CommitRefusesForeignMergeState`
  - `TestPushLock_…`'s `TeardownLockHeldPastTheWaitOwesTheRecordsPush`
  - `TestGate_Containment`'s two rows
  - `TestOpen`'s `happy_path`
  - `TestBolt_SkipGit_ShortCircuits`
  - `TrailersEndTheWeftMessageOnly`
  - `TagsEveryOrphan`
  - `TestCleanup_PrefixSetOverBareSlugOrphan`
  - `TestMergeContinue_ConcludeFailureThenRetryConcludes` and `InvisibleLandedConclude`
  - `WritesTheEntry` and `APauseHolds`
  - `TestMidMerge_…_Parked`
  - `TestMergeCrucible_AbortRefusesOnTheRecordedConcludeSHAAlone`
  - `TestMergeErrors_PinnedStrings`
  - `TestMergeStageTracked_NoMergeInProgressRefuses`
  - `RemovePath_AbsentTargetRecordsNothing`
  - `PathEmptyNAReasonWithAbsentTarget`
  - `RefusingRecordsRemoteWithholdsTheCodeSide`
  - `TestList_RejectsBareRepository`, `TestList_Errors` and `TestList_ParsesPrunable`

  `remove_siblingdirty_integration_test.go` holds the single test `TestRemove_UntrackedDriveReportIsCommittedAndArchived`.
- **Card 15, reader helpers.** Each exists with the shape the card uses:
  - `lsRemoteHeads(side, dir, opts)`
  - `committedMarker`
  - `originNamespaceAt`
  - `PendingTeardowns`
  - `ForgetOriginNamespaceForTest`
  - `dotgit.ErrRefStorage`
  - `gitrepo`'s `BranchRefs` and `TagRefs`
  - `hubforge.NewOneRepoHub`
  - `gitkit.SkipBelowGitVersion`

  `gitrepo`'s open refuses a path that is not itself a repository, as `internal/gitrepo/gogit.go:58-60` says.

Tool calls:
1. A grep for each holder test and subtest card 15 names. The context did not show those test files.
2. `TeardownReport` and a repo-wide grep for `Extend` calls, to check that card 6's list of call sites is complete. The context showed only the three sites.
3. Line counts of cards 2–4's deletions, to check the 530 figure. The context had measured only card 15.
4. A grep for the helper signatures card 15 relies on, and `gogit.go:55-62`. The context did not show them.
