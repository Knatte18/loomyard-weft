# Card 4 — fabric-test-bolt-dir-sweep

**Edit:**
- `internal/fabricengine#initTinyRepo`
- `internal/fabricengine#seedFabricConfig`
- `internal/fabricengine_test#assertBoardIsWeftWorktree`
- `internal/fabricengine#TestClassifyPaths`
- `internal/fabricengine_test#cleanupCase`
- `internal/fabricengine#TestCloneHub_CreatesHubScratchDir`
- `internal/fabricengine#TestCommitWeftAt_CommitsDirtyWorktree`
- `internal/fabricengine#TestFilterHubReserved`
- `internal/fabricengine_test#findReconcilePair`
- `internal/fabricengine_test#gitDirName`
- `internal/fabricengine#TestIsReservedHubName`
- `internal/fabricengine#TestPushSpawnCommand`
- `internal/fabricengine#TestResolveEffectiveWarpURL`
- `internal/fabricengine_test#seedRepoWideFabricConfig`
- `internal/fabricengine#TestSpecsDir`
- `internal/fabricengine#TestStencilsDir`
- `internal/fabricengine_test#TestAddRollback_AdoptedWeftBranchSurvives`
- `internal/fabricengine_test#TestAddRollback_RefusesJunctionRemovalOutsideItsWorktree`
- `internal/fabricengine_test#TestAssertRecordMatchesDiff`
- `internal/fabricengine_test#TestCleanup_ProtectsPrimaryWeftBranchAfterCheckout`
- `internal/fabricengine_test#TestCloneHub_AdoptsExistingRemoteWeftPrimaryBranch`
- `internal/fabricengine_test#TestCloneHub_AnchorCreatePath`
- `internal/fabricengine_test#TestCloneHub_BoardWorktreeOrphanBranchOnEmptyWeftRemote`
- `internal/fabricengine_test#TestCloneHub_CreatesFreshWeftPrimaryBranch`
- `internal/fabricengine_test#TestCloneHub_DerivesWarpFromBinding`
- `internal/fabricengine_test#TestCloneHub_EmptyWeftRemoteLeavesPrimaryBranchBorn`
- `internal/fabricengine_test#TestCloneHub_EmptyWeftRemoteWithForeignDefaultBranch`
- `internal/fabricengine_test#TestCloneHub_NonEmptyWeftRemoteBranchUnchanged`
- `internal/fabricengine_test#TestCloneHub_StaleFabricAnchorHardErrors`
- `internal/fabricengine_test#TestFailedPushRecord`
- `internal/fabricengine_test#TestHubContainment_AddWiresNoBoardJunction`
- `internal/fabricengine_test#TestHubContainment_CloneWiresNoBoardJunction`
- `internal/fabricengine_test#TestHubContainment_ReconcileWiresNoBoardJunction`
- `internal/fabricengine_test#TestHubLogsDir_MkdirAllIdempotentAgainstFabricCreatedDotLym`
- `internal/fabricengine_test#TestHubReserved_BoardExcludedFromPathspecRoutes`
- `internal/fabricengine_test#TestHubScratchAndLogsDir`
- `internal/fabricengine_test#TestReconcile_AddsMissingRemovesStaleNoOpsCorrect`
- `internal/fabricengine_test#TestReconcile_ConvergesAllWorktreesToRepoWidePathspec`
- `internal/fabricengine_test#TestReconcile_CorrectJunctionsAreNoOp`
- `internal/fabricengine_test#TestReconcile_EmptyDefaultPathspecRemovesOptionalJunctionKeepsStructural`
- `internal/fabricengine_test#TestReconcile_NeverRemovesReservedHubName`
- `internal/fabricengine_test#TestReconcile_PreservesUserSymlinkAtAnchor`
- `internal/fabricengine_test#TestReconcile_RefusedStaleRemovalReportsNothing`
- `internal/fabricengine_test#TestReconcile_StaleRemovalFailsClosedOnUnparseableRepoWideConfig`
- `internal/fabricengine_test#TestRefusedBefore`
- `internal/fabricengine_test#TestRegenerateHubWorkspace`
- `internal/fabricengine_test#TestRemoveRefusal_ProbeLeavesPairUntouched`
- `internal/fabricengine_test#TestRemove_RefusesForeignWorktreeWithoutDeletingIt`
- `internal/fabricengine_test#TestRemove_RefusesReservedSlugsAndLeavesThemOnDisk`
- `internal/fabricengine_test#TestRemove_SweepsAnchoredLinksOnSubpathHub`
- `internal/fabricengine_test#TestRepoWideMigratedSites_ResolveFromBoardDirWithNoPerPairConfig`
- `internal/fabricengine_test#TestSnapshotWarpSHA_PerBranchScoping`
- `internal/fabricengine_test#TestSnapshotWarpSHA_TopologicalOrderBeatsCommitDate`
- `internal/fabricengine_test#TestUnwire_PreservesUserSymlinkAtAnchor`
- `internal/fabricengine_test#TestWarpLayoutFor_FastPathMatchesResolveWorktree`
- `internal/fabricengine#TestValidateWorktreeSlug`
- `internal/fabricengine#TestWarpJunctions`

**Prosa:**
- `internal/fabricengine/livestate_doc_test.go`

**Uses:**
- `internal/fabricengine#BoltDir`
- `internal/fabricengine#BoltDirName`

**Intent:** Make fabric's tests take the Bolt directory name from the constant and name the Bolt checkout in their comments (D9: every test takes the name from the constant, a comment never names the path).

- Each `"_board"` string these members build a path or an expected message from becomes `fabricengine.BoltDirName` (or `BoltDirName` inside package `fabricengine`), and each hand join of the hub and that name becomes `fabricengine.BoltDir(hub)`; a hub fixture's own `h.BoardDir()` stays as it is.
- Each comment, in these members and in the file headers of `clone_emptyweft_integration_test.go`, `clone_hubscratch_integration_test.go`, `hubcontainment_integration_test.go`, `hubreservedroutes_test.go`, `reconcile_stale_removal_test.go`, `remove_reserved_integration_test.go` and `warplayout_test.go`, that spells `_board` names "the Bolt checkout" instead; every member of those files is listed because the header comment is outside any member.
- `livestate_doc_test.go`, which declares nothing and so is listed under Prosa by its file, has its one hit reworded: the quoted error text at line 189 elides the Bolt directory name, reading "no <Bolt directory> entry and no weft sibling", so card 8's guard needs no exemption for it.
- No assertion changes meaning: every value compared is the same string, so the suites pass as before.

**ImpactSummary:** Test-only edits across fabricengine test files; values compared are unchanged, so no production code or outcome moves.
