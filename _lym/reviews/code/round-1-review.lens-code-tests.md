```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Behaviour → pinning test**

| Behaviour | Pinned by |
|---|---|
| Linked functions keyed by import path, `Funcs` table, `MapTest.Funcs` indexes | `TestGateTestMapLinkedPackages` (`wantFuncs`, `wantTests`) |
| `Linked` hashes include a subdirectory file and leave out the nested package `linked/inner` | `TestGateTestMapLinkedPackages` (`wantLinked`) |
| Every package on the coverpkg list is recorded | Same test (coverpkg line) |
| Own `OtherFiles` hashes, testdata included | Same test (`caller.OtherFiles`) |
| Spawn verdict `OutOfProcess` | Same test (`TestCallerSpawns`) |
| Spawn verdict `Unresolved` | none |
| A test covering only linked code enters the map | Same test (`TestCallerElsewhere`) |
| Format marker: version, `Commit`, `BuiltAt`, `GoVersion`, `GOOS`, `GOARCH`, `GoModHash` | Same test (`wantFormat`) |
| A linked-only landing leaves the map fresh | Subtest `ALinkedOnlyChangeLeavesTheMapFresh` |
| Dirty linked file before the build: skip, earlier map kept | Subtest `ADirtyLinkedPackageLeavesTheMapAsItWas` |
| Linked file dirtied under the tests: skip, no map, earlier map kept | Subtest `ALinkedFileDirtiedUnderTheTestsWritesNoMap` |
| Redundancy verdicts unchanged with `Linked` and `Funcs` in a map | `TestRedundancyStep` (`beta` row; the verdict lines at 245–246 are unchanged) |
| `mapDrift` names no linked file | `TestRedundancyStep` (`beta` Drift empty, line 264; `alpha.go` hashed "stale") |
| `CoveredFuncs` naming, sorting, de-duplication, nested-package exclusion, error on a bad position | `TestCoveredFuncs` |
| `LinkedPackages` filtering and its `go list` arguments | `TestLinkedPackages` |
| `GoEnvironment` result and arguments | `TestGoEnvironment` |
| `FilesUnderDirAtRevision` listing, absent directory, bad revision | `TestHistoryReads` row |
| `CodeTreeAt` | Indirectly, through the testmap integration test |

**Findings**

1. **MEDIUM, design.** The HEAD-moved check over linked packages is not pinned. Location: `internal/gatecli/testmaptree.go:624` (`changedFiles`, `changedName`).
   - The only landing test is the existing `Lands` stand-in. It commits `landed.txt` at the module root, outside every linked package.
   - So the new loop over `trees` and the skip cause that names a linked file relative to the worktree have no failing row.
   - Evidence: `testmap_integration_test.go:164`, `appendAndCommit(filepath.Dir(dir), "landed.txt")`.
   - Also in this area: `testmap_integration_test.go:695` onward has no landing subtest.
   - Fix: add a landing row to `TestGateTestMapLinkedPackages` that commits a change to a linked file during a test.

2. **MEDIUM, design.** The rule for which tracked files a package owns has no direct row for its nested-module and non-buildable-directory branches. Location: `internal/gatecli/testmaptree.go:544` (`nestedPackageDirs`, `buildableDir`).
   - The fixture only has a nested package marked by a `.go` file (`linked/inner`).
   - A nested `go.mod` is not tested.
   - A `.go` file under `testdata`, `_x` or `.x` that must not make a nested directory is not tested.
   - A `testdata` directory holding only a `.go` file (the case the summary names) is not tested either.
   - Breaking these branches would pass every test.
   - Fix: add table rows for `nestedPackageDirs`, or fixture files in the linked package, for each of these.

3. **MEDIUM, design.** A failed-build map carrying the format marker is not asserted. The record says it must carry the marker and `Commit`/`BuiltAt`. Location: `internal/gatecli/testmap_integration_test.go:559`.
   - The `broken` assertions check only commit, tests and failure text, and nothing checks `Format` on a failed build.
   - A failed-build map with `Funcs` nil and `Linked` set is also unchecked.
   - Fix: assert `Format` (version, `Commit`, `BuiltAt`) and `Funcs == nil` on the `broken` map.

4. **LOW, design.** Parts of the environment stamp are ignored by the test, so a regression in them would pass. Location: `testmap_integration_test.go:789`.
   - Ignored: `TiersDigest` and `HubEnvDigest` via `IgnoreFields`.
   - Never exercised: `GoSumHash`, because the fixture has no `go.sum`.
   - A regression in the "before `{parallel}` substitution" property of `tiersDigest` would also pass.
   - Fix: pin that the digests are non-empty and stable across a parallel-count change, and add a `go.sum` to the fixture.

5. **LOW, design.** The "fresh read" property of the pre-build dirty check is not pinned. Location: `testmap_integration_test.go:769`.
   - The dirty file already exists before the call starts, so an implementation that reused the pre-walk status list would also pass.
   - The after-tests check is pinned by `Dirties`.
   - Fix: add a stand-in step that dirties a linked file after the pre-walk read and before the package's build, or accept it as covered by the after-tests row.

6. **LOW, scope.** Error and failure paths with no row. Location: `internal/gatecli/testmappackage.go:70-90` and `:379`.
   - `LinkedPackages` failure: the stand-in `go list` never fails, so the `writeFailed` and cancelled branches are untested.
   - `GoEnvironment` failure ending the walk is untested.
   - `ScanSpawns` failure writing a failed map is untested.
   - Fix: add stand-in failure env switches and rows for these. Alternatively drop the finding for the cancelled branch, which the existing cancel tests may cover.

**Test economy:** no duplicate or per-helper redundant test found among the new tests.
- `TestLinkedPackages` and `TestGoEnvironment` pin Toolchain output that the integration stand-in bypasses, so they are not redundant.
- `TestCoveredFuncs` is a per-helper test, but no public-surface test names the function keys of every nesting case, so it stays.
- `TestCoveredFuncs` is not listed in the redundancy report as contained.

**Tool calls (1):** one `grep` of `redundancy_integration_test.go` and `testmap_integration_test.go`. My context held only the diff hunks of the redundancy test, so I needed the existing `beta` Drift and `broken` map assertions.
