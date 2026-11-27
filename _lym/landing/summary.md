# fabric: add the layout and core packages, executor and snapshot reader

First step of splitting `internal/fabricengine` so that fabric's rules can be tested without git.
No verb changes behaviour: every verb still runs on today's code, and no verb calls the new executor or reader yet.

## What changes

- `internal/fabriclayout` (new) holds fabric's names, path constants, pure path functions, plain option and result
  types and the mutation record data, path classification included.
  The engine keeps the recorder that appends and logs mutations.
- `internal/fabriccore` (new) holds the snapshot, phase, action list and outcome class types, and no rule yet.
- `internal/fabricengine` gains `runRule`, the loop that drives a rule, `outcomeClassOf`,
  and `snapshotReader`, which fills the snapshot fields a phase names.
- `internal/testkit/fabrickit` (new) builds made-up snapshots.
- `scankit.AssertNoImports` fails a package that imports a banned path, standard library included.
- Callers of the moved names import `internal/fabriclayout`, which is most of the diff.
  The engine keeps no alias of a moved name.
  Engine changes stop reaching a caller's tests only where its files, its tests and everything they depend on
  import no package that imports the engine. `hubforge`, `boardkit` and `envkit` each import it.
- `PATTERN-leaf-packages` covers both new packages.
  Other PATTERN files, `docs/` and the enforcement owner maps name the new homes.

### Widened guards

- The import allowlists of `landingshed`, `reportengine`, `mergeresolve` and `hubreconcile` admit `fabriclayout`.
- The `internal/weftname` import rule admits `fabriclayout`.
- `fabriclayout` and `fabriccore` join the owners exempt from the bare weft and warp rule.
- The mutation record guard also accepts `fabriclayout.MutationRecord` as the embedded first field.
- `testOnlyCodeAllowed` gains temporary entries, listed under "To check by hand".

## Design decisions

- Both new packages are leaves.
  `fabriclayout` imports only the standard library, `lymdirs` and `weftname`;
  `fabriccore` only the standard library and `fabriclayout`.
  Import tests enforce this and ban I/O imports.
- A rule is a pure function from a snapshot to an `Answer`: a refusal, a request for another read phase,
  or an action list. The words are "snapshot" and "action list", not "plan".
- An action list stops at its first action that is not done, and the rule is asked again with the outcome.
- Mapping `gitrepo` and `gitexec` errors to outcome classes stays in the engine.
- A read that fails only as a transport failure is recorded for the rule to judge; any other read error is returned.
- The reader reads local refs in process, and through git when the repository's ref storage is not `files`.
  Its origin read lists branches only.

## Tests

`_test.go` lines under `fabricengine`, `fabriclayout`, `fabriccore` and `fabrickit` go from 58594 to 58300.

### Added, with the rule each guards

- `TestLeafInvariant_AllowlistOnly`, in each new package: its closed import set.
- `TestLeafInvariant_NoIO`, in each new package: its I/O import ban.
- `TestEnforcement_FabrickitLeafOnly`: the kit's closed import set.
- `TestBannedImports`: the banned-import scan.
- `TestIsMutationRecordEmbed`: which first field counts as embedding the mutation record.
- `TestHubPaths`: the literal joins of the hub path functions.
- `TestStructuralDirs`: a caller cannot change a structural directory set.
- `TestMutationsOf`: a mutation record shares no slice with its input or output.
- `TestRefusal_Error`: a refusal's message with and without a way forward.
- `TestBuilder`: each setter's placement, and that a built snapshot is its own copy.
- `TestRunRule`: the loop's phases, locks, actions, re-reads and read failures.
- `TestOutcomeClassOf`: the class of each error.
- `TestSnapshotReader_ReadsAHub`: the field reads on a real hub.
- `TestSnapshotReader_RefStorage`: the ref-storage probe, the git fallback and the worktree read.

### Moved to `fabriclayout`, with the rule each alone guards

- `TestClassifyPaths`: routing into the code, records or never-committed bucket.
- `TestScopedPathspec`: entries joined under the anchor.
- `TestRecordsBranchName`: the records branch name composition.
- `TestWeftWarpSlug`: the slug parsed from a sibling name, and the names refused.
- `TestArchiveTagSlug`: the archive tag shape the reader accepts.
- `TestOwnerArchiveTagName`: the owner archive tag name.
- `TestIsReservedHubName`: the reserved hub-entry names.
- `TestInWorktreeStaging`: the staging-directory predicate.
- `TestMutations_MarshalJSON`: the record's exact JSON.
- `TestMutationRecord_EmbedsAndMarshalsUnderMutationsKey`: an embedded record marshals under `mutations`.
- `TestOrigin_JSONRoundTrip`: the origin record's wire keys, and decoding a record without the later keys.
- `TestCommitResult_Committed`: the four answers of `CommitResult.Committed`.

### Kept in place, with the rule each guards

These lost assertions, or hold the rule of a moved name.
Apart from the widened guards, the other edited tests follow moved or renamed names and check what they did before.

- `TestFilterHubReserved`: the reserved-name filter.
- `TestWeftPathAccessors`: the path accessors mirror the anchor.
- `TestArchiveTagPushDelays`: the archive push's retry schedule.
- `TestStructuralNameSets_ForAConfigNamingNeitherStructuralDirectory`: `_lym` and `.lym` wiring, routing and
  reservation.
- `TestConfigTemplate`: the template's keys and defaults.
- `TestMergeVocabulary_ResultAndOptionsFieldsAreSideFree`: merge field names and JSON tags name no side.
- `TestMergeVocabulary_ErrorsAreSideFree`: a dirty-guard error stays side-free.
- `TestWarpLayoutFor_FastPathMatchesResolveWorktree`: the fast path equals the resolved layout.
- `TestMergeInAt_MergesTheGivenTipNotTheLocalOne`: the told tip is merged and an unknown one refused.
- `TestMergeCrucible_AbortRefusesAnAttemptWhoseConcludeLanded`: a hand-landed conclude survives an abort.
- `TestMergeState_Record`: the merge record's round trip, location and delete.
- `TestMergeState_ResetMergeSides`: the code-side-only reset.
- `TestTeardownLock_SerialisesAddDiscardAndPairPush`: add, discard and prime push beside a held lock.
- `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`: each side's push and report.
- `TestFailedPushRecord`: the record lifecycle and way forwards.
- `TestCommit_ResultFields`: the one-sided result fields.
- `TestCommitWeft_SeedsFabricArtifactsOnlyAndIsIdempotent`: the exact seeded entries.
- `TestNormalizeWarpURL`: the normalised URL for each input.
- `TestGate_ZeroValueDeclarationsAreRefusals`: an omitted declaration refuses.
- `TestGate_AbsentTargetIsNoOp`: an absent target passes the check.
- `TestOwnership_WarpCheckoutKind`: any worktree of the repository is accepted.
- `TestWireJunctions_ScratchLinkRefusedAfterClassify`: the warning line for each refusal.
- `TestWarpJunctionsHere`: agreement with `WarpJunctions`.
- `TestWarpJunctions`: one record per name.
- `TestWarpLymLinkHere`: the anchored `_lym` link.

### Deleted

Held by another test:

- `TestList_RefStorage`: `TestSnapshotReader_RefStorage`.
- `TestMergeContinue_FreshHandle_RecoversCrashedAfterCleanStaging`: the conclude-retry and invisible-conclude tests
  beside it, though neither builds a hand-saved staged record.
- `TestDropHandOverMark`: `TestTakeOverRun`.
- `TestCommitWeftAt_SkipGitReturnsImmediately`: `TestBolt_SkipGit_ShortCircuits`.
- `TestCommit_TwoSided_WarpFirstOrdering`: `TestCommitWithTrailers`.
- `TestRemove_UntrackedDriveReportIsCommittedAndArchived`: `TestRemove_CommitsPendingRecordsIntoArchiveTag`.
- `TestCleanup_RemoteTrueDeletesLocalAndRemoteOrphan`: `TestCleanup_ApplyArchivesEachOrphanBeforeDeleting` and
  `TestCleanup_PrefixSetOverBareSlugOrphan`.
- `TestGate_CheckOrdering`: `TestGate_Containment`.
- `TestGate_DirtinessNAEmptyReason`: `TestGate_ZeroValueDeclarationsAreRefusals`.
- `TestNew_HappyPath`: `TestOpen`; nothing now checks the handle's two fields are set.
- `TestRecordsBranchName_RoundTripsWithWeftWarpSlug`: `TestRecordsBranchName` and `TestWeftWarpSlug`.
- `TestClassifyPaths_PartitionsInputWithNothingLostOrDuplicated`: `TestClassifyPaths`.
- `TestBoardDir`, `TestHubPath`, `TestSpecsDir`, `TestStencilsDir`, `TestHubScratchAndLogsDir`: `TestHubPaths`.
- `TestHubReserved_BoardExcludedFromPathspecRoutes`: `TestConfigTemplate` and `TestFilterHubReserved`.
- `TestHubReservedNames_StillReturnsExactlyTheThreeHubStructuralTokens`: `TestIsReservedHubName`.

Guarding no rule of their own:

- `TestCheck_HasNoFourthMemberForForce`: measured a slice literal it built itself.
- `TestWorktreePath` and `TestOriginRecordPaths`: each expected value was the join under test.
- `TestHubLogsDir_MkdirAllIdempotentAgainstFabricCreatedDotLym`: tested `os.MkdirAll`.

`Loosened-Check:` commit trailers record only some of these deletions and trims,
among them the removed checks that no other test file holds.

## To check by hand

- **Widened guards**: confirm that each admission listed above is wanted.

- **Test-only functions in production files** (`internal/fabricengine/executor.go`, `snapshotread.go`,
  `internal/fabriccore/answer.go`).
  `PATTERN-test-only-code` forbids them, and its text is unchanged.
  `runRule`, `outcomeClassOf`, `fieldReadError.Error` and `Unwrap`, `snapshotReader.read`, `readLocalRefs`
  and `Refusal.Error` are allowed on purpose until a later change adds their first production caller.
  Each has a temporary `testOnlyCodeAllowed` entry in `cmd/lym/testonlycode_linux_test.go`.
- **Fabric's test code barely shrinks**: 294 of 58594 lines, by the count above.
  The aim is to cut it substantially, so judge whether more should go.
- **Deleted checks that no other test holds** (`destroy_test.go` and the other trimmed engine tests):
  read the `Loosened-Check:` trailers against the diff.
  For example, nothing checks that `removePath` creates nothing on an absent target,
  or reads the told tip back from a conflicted `MergeInAt` record.
  The no-mutations check of a refused pair push is also gone, as the refusal returns an empty result before any push.
  The registered-linked ownership refusal is held by `TestOwnership_RegisteredLinkedWorktreeKind`.
- **Executor and reader branches that lacked a test** (`executor_test.go`, `snapshotread_integration_test.go`):
  a transport failure on a re-read, a joined error mixing transport and other failures,
  one failing store beside good ones, and the records store's prefixes and absent directory.
  Cases now cover them, but none was run against the defect put back into the code.
- **Executor tests and other modules' errors** (`executor_test.go`): `PATTERN-test-stand-ins` bars made-up
  `gitexec` or `gitrepo` errors. Confirm that `TestRunRule` and `TestOutcomeClassOf` provoke them in the real module.
- The slowest test suites, those needing a tmux server or a live LLM session, were not run.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
