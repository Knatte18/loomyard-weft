# Card 3 — layout-hub-paths

**Move:**
- `internal/fabricengine#BoardWriteLockFile`
- `internal/fabricengine#BoardPushLockFile`
- `internal/fabricengine#BoltDirName`
- `internal/fabricengine#BoardDirName`
- `internal/fabricengine#HubSuffix`
- `internal/fabricengine#BoltDir`
- `internal/fabricengine#BoardDir`
- `internal/fabricengine#stencilsDirName`
- `internal/fabricengine#StencilsDir`
- `internal/fabricengine#specsDirName`
- `internal/fabricengine#SpecsDir`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#AdoptScratchDir`
- `internal/fabricengine#ClaudeAccountsDir`
- `internal/fabricengine#FrontTaskScratchDir`
- `internal/fabricengine#HubLogsDir`
- `internal/fabricengine#LogArchiveDir`
- `internal/fabricengine#RunProcDir`
- `internal/fabricengine#HubPath`
- `internal/fabricengine#portalsDirName`
- `internal/fabricengine#launchersDirName`
- `internal/fabricengine#HubReservedNames`
- `internal/fabricengine#IsReservedHubName`
- `internal/fabricengine#orphanFirstSightFileName`
- `internal/fabricengine#OrphanFirstSightPath`
- `internal/fabricengine#TeardownLockPath`
- `internal/fabricengine#HandOverMarkPath`
- `internal/fabricengine#StencilsSubtreeRel`
- `internal/fabricengine#SpecsSubtreeRel`
- `internal/fabricengine#RecordsLockDirPath`
- `internal/fabricengine#moveRecordFile`
- `internal/fabricengine#MoveRecordPath`
- `internal/fabricengine#originRecordDirName`
- `internal/fabricengine#originRecordFileName`
- `internal/fabricengine#OriginRecordRel`
- `internal/fabricengine#InWorktreeStaging`
- `internal/fabricengine#TestInWorktreeStaging`

**Create:**
- `plan:internal/fabriclayout#WorktreeStagingPrefix` -> `const WorktreeStagingPrefix = ".fabric-wt-staging-"`
- `plan:internal/fabriclayout#RecordsLockDirName` -> `const RecordsLockDirName = ".weft"`
- `plan:internal/fabriclayout#TeardownLockDirName` -> `const TeardownLockDirName = "teardown"`
- `plan:internal/fabriclayout#TestHubPaths` -> `func TestHubPaths(t *testing.T)`
- `plan:internal/fabriclayout#TestIsReservedHubName` -> `func TestIsReservedHubName(t *testing.T)`

**Delete:**
- `internal/fabricengine#weftLockDirName`
- `internal/fabricengine#teardownLockDirName`
- `internal/fabricengine#worktreeStagingPrefix`
- `internal/fabricengine#TestBoardDir`
- `internal/fabricengine#TestHubPath`
- `internal/fabricengine#TestSpecsDir`
- `internal/fabricengine#TestStencilsDir`
- `internal/fabricengine#TestIsReservedHubName`
- `internal/fabricengine#TestHubReservedNames_StillReturnsExactlyTheThreeHubStructuralTokens`

**Edit:**
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#restoreWeftLockDir`
- `internal/fabricengine#seedWeftArtifactExcludes`
- `internal/fabricengine#PendingTeardowns`
- `internal/fabricengine#StencilBaseByStamp`
- `internal/fabricengine#WeftLockDirNameForTest`
- `internal/fabricengine#mkWorktreeStagingDir`
- `internal/lymcwd#TestEnforcement_GeometryLiterals`
- `internal/fabricengine/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/fabriclayout#StructuralCommittedDirs`
- `plan:internal/fabriclayout#StructuralNeverCommittedDirs`
- `internal/fabricengine#containsName`

**Intent:** Move the hub's path names and pure path functions into `internal/fabriclayout`'s `hubpaths.go` (D1, D7), and retarget every caller.

The Move targets go to `internal/fabriclayout/hubpaths.go` with their doc comments, bodies unchanged but for these:
- `BoltDirName` is `lymdirs.BoltDirName`, since the layout cannot import `lymcwd` (D1); the value stays `_bolt`.
- `TeardownLockPath` joins `TeardownLockDirName`, and `RecordsLockDirPath` joins `RecordsLockDirName`;
  these two exported constants replace the engine's `teardownLockDirName` and `weftLockDirName`, which the engine still reads.
- `IsReservedHubName` reads `StructuralCommittedDirs()` and `StructuralNeverCommittedDirs()`.
- `InWorktreeStaging` reads `WorktreeStagingPrefix`, the exported constant that replaces `destroy.go`'s `worktreeStagingPrefix`;
  `mkWorktreeStagingDir`, which mints the staging names, reads `fabriclayout.WorktreeStagingPrefix`, so the prefix stays one constant.
  `TestInWorktreeStaging` (tier0) moves to the layout with its rows, reading `WorktreeStagingPrefix`.

Every reference to a moved name, in `internal/fabricengine` and in every other production and test file, becomes `fabriclayout.<Name>`.
A file whose only fabric names are moved ones then imports `internal/fabriclayout` and drops its `internal/fabricengine` import (D7);
a comment that names `fabricengine.<Name>` for a moved name names `fabriclayout.<Name>`.
The engine's remaining readers of the two directory names read the exported constants:
`ReadOriginFor`, `restoreWeftLockDir` and `seedWeftArtifactExcludes` read `fabriclayout.RecordsLockDirName`, keeping their message text byte for byte;
`PendingTeardowns` reads `fabriclayout.TeardownLockDirName`;
`WeftLockDirNameForTest` is `fabriclayout.RecordsLockDirName`.
`StencilBaseByStamp` joins `fabriclayout.StencilsSubtreeRel()` with the stencil's path, the same path it builds today.

`internal/lymcwd#TestEnforcement_GeometryLiterals`' `geometryTokenOwners` names `internal/fabriclayout` where it named `internal/fabricengine`:
`-LYMHUB` is owned by `internal/lymcwd` and `internal/fabriclayout`, `_portals` and `_launchers` by `internal/fabriclayout`.
Its comments say the layout package owns `HubPath` and the reserved names (D7).

Tests shrink:
- `TestHubPaths` (tier0, table) has one row per moved path function, the row's want a literal path such as `/h/_bolt/_lym/specs`.
  It replaces `TestBoardDir`, `TestHubPath`, `TestSpecsDir` and `TestStencilsDir`, deleted here.
  Each deleted test checked one join against a want the code computed; the rows pin the literal joins instead.
  `specsdir_test.go` and `stencilsdir_test.go` are left empty by the deletions and are removed.
- `TestIsReservedHubName` (tier0) is today's `TestIsReservedHubName` table, moved; its comment naming `structuralCommittedDirs` and `structuralNeverCommittedDirs` names `StructuralCommittedDirs()` and `StructuralNeverCommittedDirs()`.
  It takes rows for `HubReservedNames`' exact members and its lack of `.lym`, which `TestHubReservedNames_StillReturnsExactlyTheThreeHubStructuralTokens` asserted; that test is deleted.
  `containsName` stays in `structuraldirs_test.go` for the tests that still call it.
- `TestFilterHubReserved`, `TestStructuralNameSets_ForAConfigNamingNeitherStructuralDirectory` and `TestDeployedLymPathspec_YieldsNoDuplicateLym` stay in the engine: they test `filterHubReserved`, `pathspecNames` and `junctionNames`, which stay.

Docs: `internal/fabricengine/doc.go` names `fabriclayout.TeardownLockPath(hub, slug)` in its teardown-lock sentence.
`docs/overview.md` names `fabriclayout.BoltDir`, `fabriclayout.HubReservedNames` and `fabriclayout.IsReservedHubName`.

No behavior changes (D5): every path, lock name and reserved-name answer is the same string as before.

**ImpactSummary:** The most imported fabric names move: every importer that names a hub path retargets mechanically, most dropping the engine import; inside the engine five readers of two directory names and one geometry owner map change.

**Commit:** `3: layout-hub-paths`
