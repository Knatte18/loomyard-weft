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
- `internal/fabriclayout#WorktreeStagingPrefix`
- `internal/fabriclayout#RecordsLockDirName`
- `internal/fabriclayout#TeardownLockDirName`
- `internal/fabriclayout#TestHubPaths`
- `internal/fabriclayout#TestIsReservedHubName`

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
- `internal/fabricengine_test#TestHubScratchAndLogsDir`
- `internal/fabricengine_test#TestHubLogsDir_MkdirAllIdempotentAgainstFabricCreatedDotLym`
- `internal/fabricengine_test#TestHubReserved_BoardExcludedFromPathspecRoutes`
- `internal/fabricengine#TestOriginRecordPaths`

**Edit:**
- `internal/fabricengine#TestFilterHubReserved`
- `internal/reedengine/doc.go`
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
- `internal/fabriclayout#StructuralCommittedDirs`
- `internal/fabriclayout#StructuralNeverCommittedDirs`
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
Its comments say the layout package owns `HubPath` and the reserved names (D7),
and the `_bolt` row's comment says `lymdirs.BoltDirName` is the one declaration, which the layout package and `runproc` take the name from.

Tests shrink:
- `TestHubPaths` (tier0, table) has one row per moved path function, each want built from `lymdirs.BoltDirName` concatenated with literal segments and passed through `filepath.FromSlash`, such as `filepath.FromSlash("/h/" + lymdirs.BoltDirName + "/_lym/specs")`, as card 2's `TestClassifyPaths` row spells the Bolt directory.
  No want holds a literal `_bolt`: `TestEnforcement_BoltDirNameDeclaredOnce` fails every whole-token spelling of the Bolt directory name in any Go file but its declaration.
  `FromSlash` gives each want the separator `filepath.Join` produces, so the rows hold on Windows too.
  It replaces `TestBoardDir`, `TestHubPath`, `TestSpecsDir`, `TestStencilsDir` and `TestHubScratchAndLogsDir`, deleted here.
  Each deleted test checked joins against wants the code computed; the rows pin the literal joins instead.
  `TestHubScratchAndLogsDir`' rule that the scratch directory is a sibling of the stencils directory's `_lym` component, not nested under it, is held by the `HubScratchDir` row's want, `filepath.FromSlash("/h/" + lymdirs.BoltDirName + "/.lym")`, beside the `StencilsDir` row's `filepath.FromSlash("/h/" + lymdirs.BoltDirName + "/_lym/stencils")`.
  `specsdir_test.go` and `stencilsdir_test.go` are left empty by the deletions and are removed.
- `TestHubLogsDir_MkdirAllIdempotentAgainstFabricCreatedDotLym` is deleted: it checks that `os.MkdirAll` succeeds twice over an existing directory, the standard library's contract, and guards no rule of fabric's.
  `hubscratch_test.go` is left empty and is removed.
- `TestHubReserved_BoardExcludedFromPathspecRoutes` is deleted and `hubreservedroutes_test.go` removed:
  `TestConfigTemplate` pins the template's pathspec as empty, so the template's `Dirs()` add nothing to either route, and `TestFilterHubReserved` holds the filter that keeps the Bolt directory out of a configured pathspec's wired names.
  What the deletion leaves unchecked is `WiredNames`' load of the template from disk, which guards no rule of its own beside those two.
- `TestOriginRecordPaths` is deleted: each want is computed by the join the accessors run, so it asserts nothing the code does not compute itself.
- `TestIsReservedHubName` (tier0) is today's `TestIsReservedHubName` table, moved; its comment naming `structuralCommittedDirs` and `structuralNeverCommittedDirs` names `StructuralCommittedDirs()` and `StructuralNeverCommittedDirs()`.
  The table takes a `junctionNames` column, so the junction-only and unrelated-name subtests become rows.
  The subtests `hub-structural tokens reserved for empty junctionNames`, `default pathspec union reserves exactly four names` and `raddle slug is no longer reserved` are cut: the table's rows already run those names with an empty `junctionNames`.
  It ends with one assertion that `HubReservedNames()` equals its literal members, which holds its lack of `.lym` too, as `TestHubReservedNames_StillReturnsExactlyTheThreeHubStructuralTokens` asserted; that test is deleted.
  A predicate row cannot hold it, since `IsReservedHubName(".lym", nil)` is true.
  `containsName` stays in `structuraldirs_test.go` for the tests that still call it.
- `TestFilterHubReserved` stays in the engine, since `filterHubReserved` stays, and loses its two loops over `HubReservedNames()`' members, which `TestIsReservedHubName`'s new assertion holds.
  The file header of `internal/fabricengine/junctionnames_test.go` then names only the tests that stay there: `junctionNames`, `filterHubReserved` and `ConfigDirs`.
- `TestStructuralNameSets_ForAConfigNamingNeitherStructuralDirectory` and `TestDeployedLymPathspec_YieldsNoDuplicateLym` stay in the engine: they test `pathspecNames` and `junctionNames`, which stay.

Docs: `internal/fabricengine/doc.go` names `fabriclayout.TeardownLockPath(hub, slug)` in its teardown-lock sentence.
`docs/overview.md` names `fabriclayout.BoltDir`, `fabriclayout.HubReservedNames` and `fabriclayout.IsReservedHubName`.
`internal/reedengine/doc.go` says the one fabric call is `fabriclayout.HubScratchDir` in `SpawnWatchdog`, and that reedengine's production files import no `internal/fabricengine`.

No behavior changes (D5): every path, lock name and reserved-name answer is the same string as before.

**ImpactSummary:** The most imported fabric names move: every importer that names a hub path retargets mechanically, most dropping the engine import; inside the engine five readers of two directory names and one geometry owner map change.

**Commit:** `3: layout-hub-paths`
