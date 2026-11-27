# Card 2 — layout-package-classify

**Create:**
- `internal/fabriclayout/doc.go`
- `plan:internal/fabriclayout#TestMain` -> `func TestMain(m *testing.M)`
- `plan:internal/fabriclayout#TestLeafInvariant_AllowlistOnly` -> `func TestLeafInvariant_AllowlistOnly(t *testing.T)`
- `plan:internal/fabriclayout#TestLeafInvariant_NoIO` -> `func TestLeafInvariant_NoIO(t *testing.T)`
- `plan:internal/fabriclayout#ClassifyPaths` -> `func ClassifyPaths(relPath string, routingNames, neverCommittedNames []string, files []string) (warp, weft, neverCommitted []string)`
- `plan:internal/fabriclayout#normalizedScopedPathspec` -> `func normalizedScopedPathspec(relPath string, names []string) []string`
- `plan:internal/fabriclayout#isUnderAnyPrefix` -> `func isUnderAnyPrefix(normalizedFile string, normalizedPrefixes []string) bool`
- `plan:internal/fabriclayout#StructuralCommittedDirs` -> `func StructuralCommittedDirs() []string`
- `plan:internal/fabriclayout#StructuralNeverCommittedDirs` -> `func StructuralNeverCommittedDirs() []string`
- `plan:internal/fabriclayout#TestClassifyPaths` -> `func TestClassifyPaths(t *testing.T)`
- `plan:internal/fabriclayout#TestStructuralDirs` -> `func TestStructuralDirs(t *testing.T)`

**Move:**
- `internal/fabricengine#ScopedPathspec`
- `internal/fabricengine_test#TestScopedPathspec`

**Delete:**
- `internal/fabricengine/classify.go`
- `internal/fabricengine/classify_test.go`
- `internal/fabricengine#structuralCommittedDirs`
- `internal/fabricengine#structuralNeverCommittedDirs`

**Edit:**
- `internal/fabricengine#Fabric.commit`
- `internal/fabricengine#pathspecNames`
- `internal/fabricengine#junctionNames`
- `internal/fabricengine#IsReservedHubName`
- `internal/fabricengine#detectWarpPollution`
- `internal/fabricengine#TestConfigTemplate`
- `internal/fabricengine#wiredNamesFromConfig`
- `internal/fabricengine#TestHubReservedNames_StillReturnsExactlyTheThreeHubStructuralTokens`
- `internal/fabricengine#TestStructuralNameSets_ForAConfigNamingNeitherStructuralDirectory`
- `internal/fabricvocab#owners`
- `internal/fabriccli#addWeftVerbs`
- `internal/fabriccli_test#TestCommand_CloneEndToEnd`
- `pattern/PATTERN-lym-files.md`
- `pattern/PATTERN-fabric-boundary.md`
- `docs/overview.md`

**Extends:**
- `pattern/PATTERN-leaf-packages.md`

**Uses:**
- `internal/fabricvocab/doc.go`
- `internal/fabriccli/doc.go`
- `plan:internal/testkit/scankit#AssertNoImports`
- `internal/testkit/scankit#AssertImportAllowlistNoStale`

**Intent:** Create the leaf package `internal/fabriclayout` (D1) and move classify.go's path classification into it, the move D9 pairs with the `PATTERN-lym-files` background edit.

The package:
- `doc.go` says the package holds fabric's names, path constants, pure path functions over absolute paths and plain strings, plain option and result types and the mutation record data;
  that it imports only the standard library, `internal/lymdirs` and `internal/weftname` and runs no git, starts no process, touches no filesystem and takes no lock;
  that a function touching the filesystem or git, and an option type with a seam or callback field, stays in `internal/fabricengine`;
  and that a file outside fabric that uses only these names imports this package and not the engine, so an engine change leaves it out of the gate scope (D7).
  It documents only this package.
- `TestMain` runs the package's tests under `tmuxkit.Main`, as every package's `TestMain` does (`PATTERN-test-isolation`).
- `TestLeafInvariant_AllowlistOnly` (tier0, `//lym:guard`, `//lym:watch internal/fabriclayout/*.go internal/testkit/scankit/*.go`) calls `scankit.AssertImportAllowlistNoStale` with `internal/lymdirs` alone;
  card 4 adds `internal/weftname` when the first file imports it.
  It fails on any import of `gitexec`, `gitrepo`, `dotgit`, `lymcwd` or any other repository package.
- `TestLeafInvariant_NoIO` (tier0, same markers) calls `AssertNoImports` with `os`, `os/...`, `io/ioutil`, `io/fs`, `syscall`, `net` and `net/...`, so a standard-library import that reaches the disk, a process or the network fails it.
  The scan sees imports only: a `path/filepath` call that reads the disk, such as `Abs`, `EvalSymlinks`, `Glob` or `Walk`, is held by review, and the background below states that bound.

The classification:
- `ClassifyPaths` is today's `classifyPaths` body, exported, its doc comment kept; `normalizedScopedPathspec` and `isUnderAnyPrefix` are today's helpers, the latter renamed from `isUnderAnyWeftPrefix`.
- `StructuralCommittedDirs` returns a fresh `[]string{lymdirs.LymDirName}` and `StructuralNeverCommittedDirs` a fresh `[]string{lymdirs.DotLymDirName}`, each carrying the doc comment of the var it replaces.
  A function, not an exported var, so no caller can change the structural sets.
- `ScopedPathspec` moves unchanged from `fabric.go`; every caller, in the engine and outside it, now calls `fabriclayout.ScopedPathspec`.
  `TestScopedPathspec` (tier0) moves from `fabric_test.go` to the layout's tests unchanged.
- `classify.go` and `classify_test.go` go; the engine's callers name the new functions:
  `Fabric.commit` calls `fabriclayout.ClassifyPaths` with `fabriclayout.StructuralNeverCommittedDirs()`;
  `pathspecNames`, `junctionNames`, `IsReservedHubName` and `detectWarpPollution` read the two sets through the functions;
  so do `TestConfigTemplate` and `wiredNamesFromConfig`.
  `TestHubReservedNames_StillReturnsExactlyTheThreeHubStructuralTokens` compares with `slices.Equal`, since `stringSlicesEqual` leaves with `classify_test.go`.
- `TestClassifyPaths` (tier0) holds today's `TestClassifyPaths` rows with their literal expectations and its `//testtiming:keep` reason.
  The row `routingNames_with_more_than_two_entries` spells the Bolt directory as `lymdirs.BoltDirName`, since the engine's `BoltDirName` is out of the layout's reach.
  `TestClassifyPaths_PartitionsInputWithNothingLostOrDuplicated` is deleted with `classify_test.go` and has no successor:
  each `TestClassifyPaths` row checks all three buckets for exact, ordered equality with literals that partition its input, so the partition test asserts nothing those rows do not.
- `TestStructuralDirs` (tier0) holds one row: overwriting a returned slice's first element and appending to it leaves the next call's answer unchanged, the guarantee the functions exist for.
  The overwrite is what fails a shared package slice, since an append to a one-element slice with no spare capacity always allocates.
  It pins no literal member: `TestIsReservedHubName`'s `_lym` row and `TestStructuralNameSets_ForAConfigNamingNeitherStructuralDirectory`'s `.lym` checks already hold the members.
- The comment of `TestStructuralNameSets_ForAConfigNamingNeitherStructuralDirectory` and the file header of `internal/fabricengine/structuraldirs_test.go` name `fabriclayout.ClassifyPaths`, `fabriclayout.StructuralCommittedDirs` and `fabriclayout.StructuralNeverCommittedDirs` where they named the unexported names.

Vocabulary and docs:
- `internal/fabricvocab#owners` gains `internal/fabriclayout`, so its names may carry the side tokens (the record's `PATTERN-fabric-boundary` constraint).
- `pattern/PATTERN-fabric-boundary.md`: the owner set line gains `fabriclayout`;
  the weft-commits bullets name `fabriclayout.ScopedPathspec`, `fabriclayout.StructuralNeverCommittedDirs` and `fabriclayout.ClassifyPaths`.
- `pattern/PATTERN-lym-files.md`'s structural-split bullet names `fabriclayout.StructuralCommittedDirs` and `StructuralNeverCommittedDirs` as the home of the split (D9).
- `pattern/PATTERN-leaf-packages.md` gains an `internal/fabriclayout` section:
  "Imports only the standard library, `internal/lymdirs` and `internal/weftname`." then
  "It runs no git, starts no process, touches no filesystem and takes no lock: no `os`, `io/ioutil`, `io/fs`, `syscall` or `net` import, nor any import under `os/` or `net/`." then
  "Bound: the scan sees imports only; a `path/filepath` call that reads the disk, such as `Abs` or `EvalSymlinks`, is held by review." then
  "It holds fabric's names, path constants, pure path functions, plain option and result types and the mutation record data, so a caller that needs only these leaves the engine's gate scope."
- `docs/overview.md`'s two sentences naming `structuralNeverCommittedDirs` and `structuralCommittedDirs` name the `fabriclayout` functions.
- The comments in `addWeftVerbs` and `TestCommand_CloneEndToEnd` that name `structuralNeverCommittedDirs` name `fabriclayout.StructuralNeverCommittedDirs`.

No behavior changes (D5): every verb classifies, refuses and commits as before.

**ImpactSummary:** One new leaf package with three exported functions plus a moved `ScopedPathspec`; the engine's callers of the classification and the structural sets are local to `internal/fabricengine`, `ScopedPathspec`'s outside callers take a qualifier change, and three docs follow the new names.

**Commit:** `2: layout-package-classify`
