# Card 4 — layout-ref-names

**Move:**
- `internal/fabricengine/branchname.go`
- `internal/fabricengine_test#TestRecordsBranchName`
- `internal/fabricengine#WeftWarpSlug`
- `internal/fabricengine#TestWeftWarpSlug`
- `internal/fabricengine#ArchiveTagSHALen`
- `internal/fabricengine#ArchiveTagPrefix`
- `internal/fabricengine#ArchiveTagSlug`
- `internal/fabricengine#ArchiveTagName`
- `internal/fabricengine#TestArchiveTagSlug`
- `internal/fabricengine#OwnerArchiveTagPrefix`
- `internal/fabricengine#OwnerArchiveTagName`
- `internal/fabricengine#TestOwnerArchiveTagName`
- `internal/fabricengine#OwnerArchiveSideCode`
- `internal/fabricengine#OwnerArchiveSideRecords`
- `internal/fabricengine#RecordsNamespaceRoot`
- `internal/fabricengine#RecordsNamespaceHeads`
- `internal/fabricengine#RecordsNamespaceTags`
- `internal/fabricengine#RecordsNamespaceCodeTags`
- `internal/fabricengine#OneRepositoryBoardBranch`
- `internal/fabricengine#PrimeRecordsBranchKey`
- `internal/fabricengine#WarpBindingFileName`
- `internal/fabricengine#ShortnameFileName`
- `internal/fabricengine#WarpSHATrailerKey`
- `internal/fabricengine#TransitionTrailerKey`
- `internal/fabricengine#SnapshotTrailerKey`
- `internal/fabricengine#originRecordSubjectPrefix`
- `internal/fabricengine#OriginRecordSubject`
- `internal/fabricengine#OriginRecordSlug`
- `internal/fabricengine#DefaultCommitMessage`
- `internal/fabricengine#PauseSlugFlag`
- `internal/fabricengine#TeardownSlugFlag`
- `internal/fabricengine#ParentLockedFlag`
- `internal/fabricengine#TeardownChildVerb`
- `internal/fabricengine#TeardownChildSlugFlag`
- `internal/fabricengine#TeardownChildTryFlag`
- `internal/fabricengine#TeardownChildWaitFlag`
- `internal/fabricengine#MergeVerbMergeIn`
- `internal/fabricengine#MergeVerbMerge`
- `internal/fabricengine#MoveKindMove`
- `internal/fabricengine#MoveKindIntegrate`
- `internal/fabricengine#FabricReaderVerbs`
- `internal/fabricengine#SeedCommitMessage`
- `internal/fabricengine#SyncCommitMessage`
- `internal/fabricengine#syncSubjectPrefix`

**Create:**
- `internal/fabriclayout#SeedSubjectPrefix`

**Delete:**
- `internal/fabricengine_test#TestRecordsBranchName_RoundTripsWithWeftWarpSlug`
- `internal/fabricengine#seedSubjectPrefix`

**Edit:**
- `internal/fabriclayout#TestLeafInvariant_AllowlistOnly`
- `internal/lymcwd#weftnameImportOwners`
- `internal/lymcwd#TestEnforcement_FabricVocabulary`
- `internal/fabricengine#IsSeedCommit`
- `internal/fabricengine#TestWeftPathAccessors`
- `internal/fabricengine/doc.go`
- `pattern/PATTERN-single-repo-hub.md`

**Intent:** Move fabric's branch, tag, ref-namespace, trailer-key, flag and subject names into `internal/fabriclayout`'s `refnames.go` (D1, D7), and retarget every caller.

`branchname.go` moves whole to `internal/fabriclayout/branchname.go`, package clause changed; the other Move targets go to `internal/fabriclayout/refnames.go` with their doc comments and bodies.
Each is a constant or a pure function of strings: `RecordsBranchName` and `WeftWarpSlug` use `weftname.Suffix`, the archive and owner-archive tag functions use the moved prefixes and `ArchiveTagSHALen`, and `OriginRecordSubject` and `OriginRecordSlug` use the moved subject prefix.
`FabricReaderVerbs` is a fixed list of verb names, and `SeedCommitMessage` and `SyncCommitMessage` build commit subjects from their prefixes.
The seed prefix becomes the exported `SeedSubjectPrefix`, since the engine's `IsSeedCommit` still reads it; `syncSubjectPrefix` has no other reader and moves unexported.
What reads disk or git around them stays in the engine: `ArchiveOwnerTips`, `originNamespaceAt`, the trailer parsers, `ReadShortname` and `IsSeedCommit`, which now reads `fabriclayout.SeedSubjectPrefix`.

Every reference to a moved name, in `internal/fabricengine` and in every other production and test file, becomes `fabriclayout.<Name>`, imports following as card 3 says;
a comment naming `fabricengine.<Name>` for a moved name names `fabriclayout.<Name>`.

`internal/fabriclayout` now imports `internal/weftname`:
- `TestLeafInvariant_AllowlistOnly` admits `internal/weftname` beside `internal/lymdirs`.
- `internal/lymcwd#weftnameImportOwners` gains `internal/fabriclayout`, with a comment naming the branch-name functions it imports `weftname` for (D7).
  The map's doc comment and `TestEnforcement_FabricVocabulary`'s comment, which list the import rule's owners, name `fabriclayout` too.

Tests:
- `TestRecordsBranchName` (tier0) moves to the layout package's external test package with the emptied `branchname_test.go` removed.
- `TestRecordsBranchName_RoundTripsWithWeftWarpSlug` is deleted: `TestRecordsBranchName`'s rows and `TestWeftWarpSlug`'s rows pin both directions of the same suffix with literals, so the round trip asserts nothing they do not.
- `TestWeftWarpSlug`, `TestArchiveTagSlug` and `TestOwnerArchiveTagName` (tier0) move to `internal/fabriclayout` unchanged.
  `TestArchiveTagPushDelays` stays, since it tests the engine's push delays.
- `TestWeftPathAccessors` stays in the engine, since the accessors take a `lymcwd` location, and loses its pairing tail:
  that tail compares a base the test joins itself, and the base of `RecordsWorktreePath` it checks is held by each row's literal `wantWeftWorktreePath`.

Docs: `internal/fabricengine/doc.go`'s one-repository section says the only spelling of the records namespace is `fabriclayout`'s constants whose names open with `RecordsNamespace`.
`pattern/PATTERN-single-repo-hub.md`'s layout bullets say the namespace is spelled once, by `fabriclayout`'s `RecordsNamespaceRoot`, `RecordsNamespaceHeads` and `RecordsNamespaceTags`, and the board's branch is `fabriclayout`'s `OneRepositoryBoardBranch`.

No behavior changes (D5): every name is the same string as before.

**ImpactSummary:** Forty constants and pure name functions move; their importers retarget mechanically, the layout gains its `weftname` import, and one lymcwd import owner entry follows.

**Commit:** `4: layout-ref-names`
