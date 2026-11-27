# Card 15 — snapshot-reader

**Create:**
- `internal/fabricengine#snapshotReader`
- `internal/fabricengine#snapshotReader.read`
- `internal/fabricengine#readLocalRefs`
- `internal/fabricengine#ReadSnapshotForTest`
- `internal/fabricengine#WritePendingTeardownForTest`
- `internal/fabricengine_test#TestSnapshotReader_ReadsAHub`
- `internal/fabricengine_test#TestSnapshotReader_RefStorage`

**Edit:**
- `cmd/lym#testOnlyCodeAllowed`
- `internal/fabricengine#TestConfigTemplate`
- `internal/fabricengine_test#TestWarpLayoutFor_FastPathMatchesResolveWorktree`
- `internal/fabricengine_test#TestMergeInAt_MergesTheGivenTipNotTheLocalOne`
- `internal/fabricengine_test#TestMergeCrucible_AbortRefusesAnAttemptWhoseConcludeLanded`
- `internal/fabricengine#TestMergeVocabulary_ErrorsAreSideFree`
- `internal/fabricengine_test#TestMergeState_Record`
- `internal/fabricengine_test#TestMergeState_ResetMergeSides`
- `internal/fabricengine_test#TestTeardownLock_SerialisesAddDiscardAndPairPush`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/fabricengine_test#TestFailedPushRecord`
- `internal/fabricengine_test#TestCommit_ResultFields`
- `internal/fabricengine_test#TestCommitWeft_SeedsFabricArtifactsOnlyAndIsIdempotent`
- `internal/fabricengine#TestNormalizeWarpURL`
- `internal/fabricengine#TestGate_ZeroValueDeclarationsAreRefusals`
- `internal/fabricengine#TestGate_AbsentTargetIsNoOp`
- `internal/fabricengine_test#TestOwnership_WarpCheckoutKind`
- `internal/fabricengine_test#TestWireJunctions_ScratchLinkRefusedAfterClassify`
- `internal/fabricengine#TestWarpJunctionsHere`
- `internal/fabricengine#TestWarpJunctions`
- `internal/fabricengine#TestWarpLymLinkHere`

**Delete:**
- `internal/fabricengine_test#TestList_RefStorage`
- `internal/fabricengine#TestCheck_HasNoFourthMemberForForce`
- `internal/fabricengine_test#lymcwdDotLym`
- `internal/fabricengine_test#TestMergeContinue_FreshHandle_RecoversCrashedAfterCleanStaging`
- `internal/fabricengine_test#TestDropHandOverMark`
- `internal/fabricengine#TestCommitWeftAt_SkipGitReturnsImmediately`
- `internal/fabricengine_test#TestCommit_TwoSided_WarpFirstOrdering`
- `internal/fabricengine/remove_siblingdirty_integration_test.go`
- `internal/fabricengine_test#TestCleanup_RemoteTrueDeletesLocalAndRemoteOrphan`
- `internal/fabricengine#TestGate_CheckOrdering`
- `internal/fabricengine#TestGate_DirtinessNAEmptyReason`
- `internal/fabricengine#TestWorktreePath`
- `internal/fabricengine_test#TestNew_HappyPath`
- `internal/fabricengine#NewPairedFromPathsForTest`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#snapshotRead`
- `internal/fabricengine#fieldReadError`
- `internal/fabricengine#ForgetOriginNamespaceForTest`
- `internal/fabricengine#List`
- `internal/fabricengine#lsRemoteHeads`
- `internal/fabricengine#originNamespaceAt`
- `internal/fabricengine#committedMarker`
- `internal/fabricengine#PendingTeardowns`
- `internal/fabricengine#writePendingTeardown`
- `internal/fabricengine_test#TestList_ParsesPrunable`

**Intent:** Add the phased snapshot reader (D2) in `internal/fabricengine/snapshotread.go`: the `snapshotRead` the executor loop calls, reading only the fields a phase names.
`snapshotReader` holds the hub path and each store's repository directory; `read` fills the named fields of `snapshot` and leaves the rest alone, so a verb that refuses early reads no more than it does today.

Fields and their reads, all through `gitexec`, `gitrepo` or in-process parsers (`PATTERN-gitrepo-client-boundary`):
- `FieldWorktrees` lists the code store's worktrees through `List`, which reads `dotgit.ListWorktrees` in process and falls back to `git worktree list --porcelain` on `dotgit.ErrRefStorage`, as it does today.
- `FieldLocalRefs` reads each store's branches and tags through `readLocalRefs`, keyed `refs/heads/<name>` and `refs/tags/<name>`.
  `readLocalRefs` probes its own store with `dotgit.ListWorktrees`, in process, so every store, the records and Bolt stores included, learns its ref storage with no git process.
  A `files` store is read in process through `gitrepo`'s `BranchRefs` and `TagRefs`.
  A store whose ref storage is not `files` is read through `git for-each-ref`, each annotated tag peeled to its commit and a tag naming no commit skipped, as `TagRefs` does, since go-git refuses to open such a store and so never yields a wrong ref (D2).
- `FieldOriginRefs` reads each store's origin heads through `lsRemoteHeads` with zero options, which maps a records store's names through its layout, and sets `OriginRead`.
- `FieldAnchors` lists the records store's worktrees through `List` itself and reads each one's committed marker at `HEAD` through `committedMarker`, so it needs no other field; a worktree with no committed marker has no entry.
- `FieldRecordsStore` sets `Present` when the reader holds a records store directory that exists, and `Heads` and `Tags` from `originNamespaceAt`.
  The layout is read through `originNamespaceAt`'s per-process cache, as today's verbs read it, so a later re-read of this field in one process answers the first read; the doc says so, and a rule reads it in its first phase.
- `FieldPendingTeardowns` reads `PendingTeardowns(hub)`.
A field no store or hub path answers is left empty.
Every field read returns its error as a `*fieldReadError` naming the field and the store it read, keeping the inner error's message, so `errors.Is` still finds `ErrRecordsLayout` and the message keeps the refspecs to restore.
`FieldLocalRefs` and `FieldOriginRefs`, the fields read over several stores, walk them in the fixed order `StoreCode`, `StoreRecords`, `StoreBolt`, skipping a store the reader does not hold;
a failing store does not stop the walk, so every other store is still read and filled, and the error returned joins through `errors.Join`, in walk order, one `*fieldReadError` per failing store, each carrying its `Store`.
So an unreachable code origin hides neither a reachable records origin, whose `OriginRead` is set, nor an unreachable one, which gets a read failure of its own.
`FieldWorktrees`' error carries `StoreCode`, `FieldAnchors`' and `FieldRecordsStore`'s `StoreRecords`, and `FieldPendingTeardowns`' the zero `Store`, since it reads the hub alone.

`snapshotReader.read` and `readLocalRefs` have no production caller until part 2, `fabric-core-push-origin`.
So `cmd/lym#testOnlyCodeAllowed` admits `internal/fabricengine.snapshotReader.read` and `internal/fabricengine.readLocalRefs`, as the added entry "A chain part may leave production code that only a named later part first calls" allows:
each reason names this card's tests and `fabric-core-push-origin` as the part that adds its first production caller and removes the entry in that change.
In `export_test.go`, `ReadSnapshotForTest` builds a reader and reads one phase, and `WritePendingTeardownForTest` writes a pending-teardown record through `writePendingTeardown`, for the external test package, which builds its hubs through `internal/hubforge`.

Tests, tier1:
- `TestSnapshotReader_ReadsAHub` (table) is the reader's seam test (D6), the snapshot half of the first flow's; part 2 adds the core and executor half. Rows:
  - a hub with two pairs, one of whose records branches removes its `.lym-anchor` marker in a commit, an annotated tag on the prime's records branch and a pending-teardown record planted through `WritePendingTeardownForTest`, every field read:
    it wants the prime's and the pairs' branches and their tips, the annotated tag at its commit, the worktree list, the anchor of each records worktree that keeps its marker and no entry for the one that removed it, the records store and its layout, origin's heads and the planted record;
  - the same hub read for `FieldLocalRefs` alone with only the code store named: every other field stays empty, the records store's `Present` false and no store's `OriginRead` set;
  - a one-repository hub (`hubforge.NewOneRepoHub`) whose records store's fetch refspec no longer maps the namespace: the row unsets the records store's `remote.origin.fetch`, then calls `fabricengine.ForgetOriginNamespaceForTest(h.PrimeRecords())` before the read, since the hub's build already cached the good layout for the process;
    `FieldRecordsStore` returns an error that `errors.Is` matches to `ErrRecordsLayout` and whose message names the field and the store.
- `TestSnapshotReader_RefStorage` (table) is the one fallback test D6 names, for the worktree list and the refs.
  Each repository holds a branch, a lightweight tag and an annotated tag, and is passed as both the code and the records store, so each store's own probe is reached.
  Rows read `FieldWorktrees` alone from the main worktree and from a subdirectory of a linked worktree, and `FieldLocalRefs` alone from the main worktree and from the linked worktree's root,
  since `gitrepo`'s open refuses a path that is not itself a repository:
  the `files` rows run with `PATH` emptied, so the reader starts no git process;
  the reftable rows (`git init --ref-format=reftable`, skipped below git 2.45 through `gitkit.SkipBelowGitVersion`) list the worktrees and refs through git.
  Each `FieldWorktrees` row wants the worktree list, and each `FieldLocalRefs` row the refs, the annotated tag at its commit.
  It runs serially, since it empties `PATH`, process-global state.
- `TestList_RefStorage` is deleted: its rows, the main worktree and a linked worktree's subdirectory under each backend, are `TestSnapshotReader_RefStorage`'s `FieldWorktrees` rows, which read through `List`.
  `TestList_ParsesPrunable`, `TestList_RejectsBareRepository` and `TestList_Errors` stay, each guarding a `List` case the reader tests do not reach.

Further engine tests go, so the part's fabric test code shrinks in net (the overview's decision "the foundation's tests shrink").
Each holds an assertion another named test holds, or guards no rule of its own; paths are under `internal/fabricengine/`.
Whole tests and helpers deleted:
- `TestCheck_HasNoFourthMemberForForce` (`refusalof_test.go`) checks the length of a slice literal it builds itself, so it cannot fail.
- `lymcwdDotLym` (`worktreeseam_integration_test.go`) has no reader.
- `TestMergeContinue_FreshHandle_RecoversCrashedAfterCleanStaging` (`mergein_recovery_integration_test.go`): its recovery is held by `TestMergeContinue_ConcludeFailureThenRetryConcludes`, and its fresh handle by the same file's `InvisibleLandedConclude` test.
- `TestDropHandOverMark` (`takeoverrun_integration_test.go`): `TakeOverRun` is the dropping function's one caller, and `TestTakeOverRun`'s `WritesTheEntry` and `APauseHolds` subtests hold the drop with and without a mark.
- `TestCommitWeftAt_SkipGitReturnsImmediately` (`commitweftat_test.go`) is `TestBolt_SkipGit_ShortCircuits` over the function `Bolt.Commit` calls; the Bolt test, on the public entry point, stays.
- `TestCommit_TwoSided_WarpFirstOrdering` (`commit_integration_test.go`): `TestCommitWithTrailers`' `TrailersEndTheWeftMessageOnly` subtest commits both sides through the same `commit` and asserts the two-sided result.
- `TestRemove_UntrackedDriveReportIsCommittedAndArchived`, with its file `remove_siblingdirty_integration_test.go`: `TestRemove_CommitsPendingRecordsIntoArchiveTag` runs the same no-force `Remove` and asserts the same archive content.
- `TestCleanup_RemoteTrueDeletesLocalAndRemoteOrphan` (`cleanupremote_integration_test.go`): `TestCleanup_ApplyArchivesEachOrphanBeforeDeleting`'s `TagsEveryOrphan` subtest and `TestCleanup_PrefixSetOverBareSlugOrphan` delete a local and a remote orphan and assert both gone.
- `TestGate_CheckOrdering` (`destroy_test.go`): its two subtests are the requests of `TestGate_Containment`'s `AbsoluteOutsideContainer` and `WithinContainerPassesContainment` rows.
- `TestGate_DirtinessNAEmptyReason` (`destroy_test.go`): `TestGate_ZeroValueDeclarationsAreRefusals`' `PathEmptyNAReasonWithAbsentTarget` row holds it, since the gate checks the reason before the target's existence.
- `TestWorktreePath` (`junction_test.go`) wants the join the function's body runs, so it asserts nothing the code does not compute.
- `TestNew_HappyPath` (`fabric_test.go`) and its accessor `NewPairedFromPathsForTest` (`export_test.go`): `Open` runs `newPaired`, and `TestOpen`'s `happy_path` row holds it; its non-nil checks cannot fail.
  `fabric_test.go`'s file header then names only the tests the file still holds, dropping `TestNew_HappyPath` and the `ScopedPathspec` test card 2 moves out.

Parts of tests deleted, each Edit target above:
- `TestConfigTemplate` loses its tail comparing two `dedupUnion` calls over the empty `Dirs()` its earlier assertion pins.
- `TestWarpLayoutFor_FastPathMatchesResolveWorktree` loses its `HubReservedNames` loop: the exact `WorktreeName` check after it fails the same case.
- `TestMergeInAt_MergesTheGivenTipNotTheLocalOne` loses `ConflictParksTheToldTip` and its doc line: its told tip is the local tip, its `MidMerge` checks are `TestMidMerge_FabricParkedWithConflicts_Parked`'s, and the sibling subtests hold the told tip.
- `TestMergeCrucible_AbortRefusesAnAttemptWhoseConcludeLanded` loses its `RecordedConcludeSHA` row and its doc lines: the `Invisible` row holds the moved `HEAD`, and `TestMergeCrucible_AbortRefusesOnTheRecordedConcludeSHAAlone` the recorded SHA.
- `TestMergeVocabulary_ErrorsAreSideFree` loses its `Error()` string tail and the sentence naming it: `TestMergeErrors_PinnedStrings` pins each string exactly.
- `TestMergeState_Record` loses `absent record`: `TestMergeStageTracked_NoMergeInProgressRefuses` holds the load with no record, and `TestMergeSiblings_CommitRefusesForeignMergeState` the record's absence.
- `TestMergeState_ResetMergeSides` loses the prime warp worktree subtest and its doc line: `warp side conflicted` runs the same call on the same prime and wants no error.
- `TestTeardownLock_SerialisesAddDiscardAndPairPush` loses `PairPushWhileHeldExclusiveLeavesOriginUntouched`: `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`' `TeardownLockHeldPastTheWaitOwesTheRecordsPush` holds the refusal and the unmoved origin tips, a zero wait being the same loop making one try.
  It discards the push result, so it does not hold the deleted subtest's no-mutations check;
  that check guards no rule of its own, since the refusal returns the literal `PushResult{}` before any push step runs, and the unmoved tips already fail a push that ran.
- `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` loses its inline step `failing records side withholds the code side`: its subtest `RefusingRecordsRemoteWithholdsTheCodeSide` asserts the same strings and more.
- `TestFailedPushRecord` loses its `WayForward` prime check, which the exact comparison before it already fails.
- `TestCommit_ResultFields` loses `TwoSided`, which `TestCommitWithTrailers`' two-sided subtest holds.
- `TestCommitWeft_SeedsFabricArtifactsOnlyAndIsIdempotent` loses its `retiredPatterns` loop: the exact two-entry check before it fails any retired pattern.
- `TestNormalizeWarpURL` loses its tail comparing the scp and https forms: the table's rows pin each form's literal result.
- `TestGate_ZeroValueDeclarationsAreRefusals` loses its `PathZeroOwnership` and `PathZeroDirtiness` rows, held by the `WithAbsentTarget` rows over the same check.
- `TestGate_AbsentTargetIsNoOp` loses its `removePath` tail, held by `TestGate_RecordOnlyOnObservedEffect`'s `RemovePath_AbsentTargetRecordsNothing`.
- `TestOwnership_WarpCheckoutKind` loses `DisagreesWithRegisteredLinkedKindOnMainWorktree`: `TestOwnership_RegisteredLinkedWorktreeKind`'s `RefusesMainWorktree` holds the registered-linked refusal of the main worktree, and `AcceptsPrimeWarpWorktree` the warp-checkout acceptance of it.
- `TestWireJunctions_ScratchLinkRefusedAfterClassify` loses its three `Reconcile` subtests: the warn line and `pair.Detail` carry the junction outcome's text, which the `Wire` rows hold for the same arrangements, and the same file's reconcile tests hold its passing into `Detail`.
- `TestWarpJunctionsHere` keeps only `agrees with WarpJunctions`: the `at root`, `at nested subpath` and `names ordering regressions` subtests want the production formula copied, or `WarpLymLinkHere` itself.
- `TestWarpJunctions` loses `no_unsupplied_hub_structural_names`: its length and per-index name checks fail any extra entry.
- `TestWarpLymLinkHere` loses its tail and its unused `slug` field: the tail recomputes the production formula, and the rows' literal wants hold the assertion.

`doc.go`'s executor section gains the reader: the phases, the fields each phase reads, the per-store ref-storage probe and the git fallback for a store whose ref storage is not `files`.

The commit body states the measured line totals of the part's fabric `_test.go` files at the part's base and after this card, as the overview's decision "the foundation's tests shrink" asks.

**ImpactSummary:** One new engine file reading through today's helpers with no caller-visible change; one tier1 test replaces another, two test accessors and two dead-code allowlist entries follow, and engine tests whose assertions others hold are deleted.

**Commit:** `15: snapshot-reader`
