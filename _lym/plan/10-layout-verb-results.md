# Card 10 — layout-verb-results

**Move:**
- `internal/fabricengine#AddResult`
- `internal/fabricengine#CloneOptions`
- `internal/fabricengine#DiscardResult`
- `internal/fabricengine#MergeOptions`
- `internal/fabricengine#MergeResult`
- `internal/fabricengine#StageResult`
- `internal/fabricengine#MidMergeKind`
- `internal/fabricengine#MidMergeNone`
- `internal/fabricengine#MidMergeParked`
- `internal/fabricengine#MidMergeForeign`
- `internal/fabricengine#MidMergeState`
- `internal/fabricengine#PullResult`
- `internal/fabricengine#HubPullSideKind`
- `internal/fabricengine#HubPullBoard`
- `internal/fabricengine#HubPullPrime`
- `internal/fabricengine#HubPullBeam`
- `internal/fabricengine#HubPullOutcome`
- `internal/fabricengine#HubPullMoved`
- `internal/fabricengine#HubPullUpToDate`
- `internal/fabricengine#HubPullStopped`
- `internal/fabricengine#HubPullSide`
- `internal/fabricengine#HubPullResult`
- `internal/fabricengine#HubPullResult.Stopped`
- `internal/fabricengine#RemoveStepRecordCommit`
- `internal/fabricengine#RemoveStepArchive`
- `internal/fabricengine#RemoveStepTaskWorktree`
- `internal/fabricengine#RemoveStepSiblingWorktree`
- `internal/fabricengine#RemoveStepSiblingBranch`
- `internal/fabricengine#RemoveStepTaskBranch`
- `internal/fabricengine#RemoveStepSiblingBranchOnOrigin`
- `internal/fabricengine#RemoveStepTaskBranchOnOrigin`
- `internal/fabricengine#RemoveResult`
- `internal/fabricengine#RemoteWarpBranchEntry`
- `internal/fabricengine#RemoteWarpCleanupResult`
- `internal/fabricengine#PlainArchiveTag`
- `internal/fabricengine#CheckoutResult`
- `internal/fabricengine#CleanupBranchEntry`
- `internal/fabricengine#PruneEntry`
- `internal/fabricengine#UnwireVerbResult`
- `internal/fabricengine#BoltSkip`
- `internal/fabricengine#BoltSkipFetchFailed`
- `internal/fabricengine#BoltSkipDiverged`
- `internal/fabricengine#BoltSkipDirty`
- `internal/fabricengine#BoltWriteResult`
- `internal/fabricengine#ReconcileAction`
- `internal/fabricengine#ReconcileActionWeftRecreated`
- `internal/fabricengine#ReconcileActionJunctionRepointed`
- `internal/fabricengine#ReconcileActionRawAdopted`
- `internal/fabricengine#ReconcileActionUnmanagedReported`
- `internal/fabricengine#ReconcileActionAlreadyHealthy`
- `internal/fabricengine#ReconcileActionStaleRemoved`
- `internal/fabricengine#ReconcileActionVanishedMidWalk`
- `internal/fabricengine#WarpBindingOutcome`
- `internal/fabricengine#WarpBindingOutcomePresent`
- `internal/fabricengine#WarpBindingOutcomeDiverged`
- `internal/fabricengine#WarpBindingOutcomeSkipped`
- `internal/fabricengine#ReconcilePairResult`
- `internal/fabricengine#ReconcileResult`
- `internal/fabricengine#CommitResult`
- `internal/fabricengine#CommitResult.Committed`
- `internal/fabricengine#TakeOverResult`
- `internal/fabricengine#OwnerArchive`
- `internal/fabricengine_test#TestCommitResult_Committed`

**Edit:**
- `cmd/lym#destructiveGuardMutatingResultTypes`
- `internal/fabricengine#TestMergeVocabulary_ResultAndOptionsFieldsAreSideFree`
- `pattern/PATTERN-pair-remote.md`
- `internal/fabricengine/doc.go`

**Intent:** Move the mutating verbs' plain option and result types into `internal/fabriclayout`'s `verbresults.go` (D1, D7), and retarget every caller.
Each is plain data over strings, bools, slices and the layout's own types, its first field `MutationRecord` where it has one; `CommitResult.Committed` and `HubPullResult.Stopped` are pure and move with their types.
`CloneResult` stays in the engine, since its unexported `seed` field holds the engine's clone state;
so do `AddOptions`, whose `Hold` and `BackoffSleep` are function fields, and every error type, which stays with the verb that returns it until that verb's concern moves (D9).

`cmd/lym#destructiveGuardMutatingResultTypes` names `internal/fabriclayout/verbresults.go` for every moved type:
`AddResult`, `RemoveResult`, `CheckoutResult`, `UnwireVerbResult`, `ReconcileResult`, `CommitResult`, `PullResult`, `MergeResult`, `StageResult` and `BoltWriteResult`.
The guard then still checks that each embeds the record as its first field.

`TestMergeVocabulary_ResultAndOptionsFieldsAreSideFree` reads `fabriclayout.MergeResult` and `fabriclayout.MergeOptions`, and its `//lym:seam` lines name `internal/fabriclayout#MergeResult` and `internal/fabriclayout#MergeOptions`.
`pattern/PATTERN-pair-remote.md` names `fabriclayout.RemoveResult`.
`doc.go` names the `fabriclayout` home of `PullResult`, `UnwireVerbResult.WeftContent`, `MergeResult.Committed` and `CommitResult.Committed()` where it names them as this package's own; its rules are unchanged.

Every reference to a moved name, in the engine and in every other production and test file, becomes `fabriclayout.<Name>`, imports following as card 3 says.
`TestCommitResult_Committed` (tier0) tests only the pure `CommitResult.Committed`, so it moves to the layout's tests unchanged, and the emptied `committed_test.go` is removed.
The verbs' tests stay in the engine: each asserts what a verb does, and the verbs stay.
No behavior changes (D5): every verb returns and prints the same fields.

**ImpactSummary:** Sixty-two result names move with mechanical retargets across the CLI and engine callers of fabric's verbs; one guard table, one seam test and one PATTERN background follow.

**Commit:** `10: layout-verb-results`
