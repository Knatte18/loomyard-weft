---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/testmap.go:399
    summary: A package skipped by the new before-build dirty check still counts against --max, unlike the pre-walk dirty skip the record says it matches
    fix: In mapPackages, make mapPackage report whether it skipped before building (for example a second return value, or a sentinel), and undo built++ for that case so a package skipped on the before-build dirty read does not count against --max. Keep the after-tests skip counting as today. Then change the gatecli doc.go sentence that says all the skips before a build count nothing, and the testMapCmd Long text, so they name the before-build linked-file skip as counting nothing against --max.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmaptree.go:624
    summary: The HEAD-moved check over every linked package's files has no failing test row, since the only landing stand-in commits a file outside every linked package
    fix: Add a subtest to TestGateTestMapLinkedPackages in internal/gatecli/testmap_integration_test.go, driven by a new stand-in test suffix that commits a change to a file the linked package owns while the tests run. Assert that the package is skipped with a cause naming that file relative to the worktree and that the earlier map is left as it was.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap_integration_test.go:559
    summary: No test asserts that a failed-build map carries the format marker with Commit and BuiltAt, which the record requires
    fix: In TestGateTestMap, where the broken package's failed-build map is read, assert that Format is non-nil with Version equal to testcover.MapFormatVersion, Commit equal to the map's Commit and BuiltAt equal to the map's BuiltAt, and that Funcs is nil.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmaptree.go:544
    summary: The nested-package rule has no row for a nested go.mod, or for Go files under testdata or under a dot or underscore directory that must not count as a nested package
    fix: Add fixture files under the linked package in TestGateTestMapLinkedPackages, or a table-driven test of nestedPackageDirs reached through export_test.go, with rows for a nested directory holding only go.mod (excluded from the owner), a testdata directory holding only a Go file (owned), and a Go file under a directory whose name opens with a dot or an underscore (owned). Assert the hashes in Linked or the returned directories.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap_integration_test.go:769
    summary: The before-build dirty check is not shown to be a fresh read, since the dirty linked file already exists when the call starts
    fix: Add a subtest to TestGateTestMapLinkedPackages in which a test of one package writes an untracked file into a linked package, and a package that sorts later and links it is walked afterwards in the same call. Assert that the later package is skipped with the before-build cause, without the after its tests suffix, and that its earlier map is left. The check then fails for an implementation that reuses the status read taken before the walk.
  - id: F6
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/gatecli/testmappackage.go:70
    summary: The LinkedPackages failure, the GoEnvironment failure, the ScanSpawns failure and the Unresolved spawn verdict have no test row
    fix: Add stand-in switches in testmap_integration_test.go that make go list fail and go env fail, and rows asserting that a go list failure writes a failed map carrying the Format marker and that a go env failure ends the walk. Add a ScanSpawns failure row if a fixture can cause one cheaply, and a fixture test that the spawn scan marks Unresolved, asserted in the recorded MapTest.Spawn.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap_integration_test.go:789
    summary: The test ignores TiersDigest and HubEnvDigest and the fixture has no go.sum, so a regression in the stamp's digests or in GoSumHash passes
    fix: In TestGateTestMapLinkedPackages add a go.sum to the fixture and assert GoSumHash equals its content hash. Assert that TiersDigest and HubEnvDigest are non-empty, and that a second call with a different parallel count in a tier command leaves TiersDigest as it was.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/testmaptree.go:544
    summary: nestedPackageDirs counts any buildable directory holding a Go file as a package, so a directory whose Go files build constraints all exclude is cut out of the owner, unlike impactset's go list based rule
    fix: Add a sentence to the doc comment of nestedPackageDirs stating that a directory whose Go files are all excluded by build constraints still counts as nested, a bound that only misses files and never adds a false one. Alternatively derive the nested directories from the module's go list package directories.
---

### [MEDIUM] A before-build dirty skip counts against --max

**Location:** `internal/gatecli/testmap.go:399`, `internal/gatecli/testmappackage.go:67`
**Issue:** `mapPackages` does `built++` before it calls `mapPackage`, and the new first dirty read sits inside `mapPackage`.
A package skipped on a dirty linked file therefore uses a cap slot, so a dirty widely linked file can leave later stale packages unbuilt.
The record says testmap skips such a package "as it does today for a dirty file of its own directory", and that skip counts nothing; the gatecli `doc.go` says the same of the skips before a build.

### [MEDIUM] The HEAD-moved check over linked packages is unpinned

**Location:** `internal/gatecli/testmaptree.go:624`
**Issue:** The record's D2 says the check for files changed while the tests ran covers every linked package's files.
The only landing stand-in, `Lands`, commits `landed.txt` at the module root, outside every linked package, so the loop over `trees` and the skip cause that names a linked file relative to the worktree would pass any test run if broken.

### [MEDIUM] A failed-build map's format marker is unasserted

**Location:** `internal/gatecli/testmap_integration_test.go:559`
**Issue:** The record requires a failed-build map to carry the format marker with the map's `Commit` and `BuiltAt`.
The `broken` map assertions check the commit, the tests and the failure text only.

### [MEDIUM] The nested-package rule's branches are unpinned

**Location:** `internal/gatecli/testmaptree.go:544`
**Issue:** The fixture holds one nested package, `linked/inner`, marked by a Go file.
A nested `go.mod`, a Go file under `testdata`, and a Go file under a dot or underscore directory have no row, so breaking each branch passes every test.

### [MEDIUM] The before-build check is not shown to be a fresh read

**Location:** `internal/gatecli/testmap_integration_test.go:769`
**Issue:** The record says neither dirty check reuses the status list read before the walk.
In `ADirtyLinkedPackageLeavesTheMapAsItWas` the dirty file exists before the call starts, so an implementation that reused the earlier read would also pass.

### [LOW] Failure paths and the Unresolved verdict have no row

**Location:** `internal/gatecli/testmappackage.go:70`
**Issue:** The stand-in `go list` and `go env` never fail, and no fixture causes a spawn scan failure or an `Unresolved` verdict.
The `writeFailed` branch of a `LinkedPackages` failure, the walk-ending `GoEnvironment` failure and the `Unresolved` field of `MapTest.Spawn` run in no test.

### [LOW] Digests and GoSumHash of the environment stamp are unpinned

**Location:** `internal/gatecli/testmap_integration_test.go:789`
**Issue:** The test ignores `TiersDigest` and `HubEnvDigest`, and the fixture has no `go.sum`.
The property that `TiersDigest` is taken before the `{parallel}` substitution has no row.

### [LOW] The nested-package rule departs from impactset's package set

**Location:** `internal/gatecli/testmaptree.go:544`
**Issue:** impactset maps a non-Go file to the nearest package directory `go list` reports.
`nestedPackageDirs` takes any buildable directory holding a Go file, so one whose Go files build constraints all exclude is cut out of the owner here but not in impactset.
The effect is a missed hash and never a false one.

## Rejected

- Lens code-tests: `CodeTreeAt` has no direct test. It is a thin wrapper pinned through the testmap integration test and the `TestHistoryReads` row, so no finding.
- Lens code-correctness: `tiersDigest` and the redundancy `JudgeAgainst` handling of an empty kept run were marked not verified. These were not defects, and the redundancy assertions in `TestRedundancyStep` stand unchanged.
- Lens code-correctness: `ScanSpawns` failing before the dirty checks writes a failed map. This is an acceptable ordering, since the scan reads no worktree status.
- Lens code-correctness: an untracked file in a new subpackage counts as dirty for the enclosing package. This is conservative and the change description states it.
