# gate testmap: record test coverage across every linked in-module package

`lym gate testmap` builds each package's coverage binary with a coverpkg naming every in-module package its test binary links.
Each map now records which functions of those packages each test covers.
Until now a map held only the package's own blocks, so it could not tell which test of an importing package reaches a changed package.
This is the map side of gate scoping by coverage map; selecting tests from these maps is a later task (`coverage-test-select`).

The decision record's D2 was amended before the build to carry D13 of the `testmap-upkeep-design` Sketch.
Another package's coverage is stored as function sets, not block keys.
The map also records the hashes of its own package's non-Go files.
The format marker also holds D6's environment stamp.

## What a map records now

- `Linked`: for each other linked package, keyed by import path, the content hash of every file the package owns.
  A package owns every tracked file under its directory down to the next package directory or nested module.
  Files in subdirectories are included, which covers `go:embed` targets.
  Every package of the coverpkg list is recorded, whether or not a test covered it.
- `OtherFiles`: the hashes of the package's own files other than the Go files `Files` already holds, `testdata` included.
- `Funcs`: a sorted table of `FuncKey{Package, File, Func}`, naming a linked function the way a block key names a function.
  Each `MapTest.Funcs` indexes into it.
  No block of another package is stored.
- `MapTest.Spawn`: the `testcover.ScanSpawns` verdict (out-of-process or unresolved) for each test.
  Spawn-marked tests are still run and measured.
- `Format`: a `MapFormat` marker with `Version` (`MapFormatVersion = 2`), the map's `Commit` and `BuiltAt`, and an `EnvironmentStamp`.
  The stamp holds the Go version, `GOOS` and `GOARCH` (from `go env`), the hashes of `go.mod` and `go.sum` at the commit,
  a digest of the mapped tiers' commands and env before `{parallel}` substitution, and a digest of the hub's `env.yaml` variables.
  Failed-build maps carry the marker too.
  An older binary keeps the unknown `format` key when it rewrites a map, so a marker whose `Commit` or `BuiltAt` differs from the map's own shows that rewrite.
- A test that covers no block of its own package but some linked function now enters the map.

All the new fields are additions, and reads still ignore unknown keys (PATTERN-tolerant-records).

## Dirty checks

testmap still runs the existing pre-walk check for a dirty Go file in the package's own directory.
On top of that, each package it builds gets two fresh `Fabric.Status` reads.
Both check every file any linked package owns, the package's own files included:

- immediately before the build: a dirty file skips the package with `modified or untracked file <path>`;
- after the last test: a dirty file writes no map, skips with `... after its tests`, and leaves any earlier map as it was.

The existing check for HEAD moving during the tests now compares every linked package's files.
Own files are named relative to the package and linked files relative to the worktree.
The record's Bound still holds: an edit made after the first read and reverted before the second goes unseen.

## What stays the same

- Staleness is unchanged: own Go files, the age limit, or a failed build at another HEAD.
  A change only to a linked package leaves the map fresh, as the interim rule of the upkeep design's D1 requires.
- The redundancy report keeps its own-package fresh builds, reads only `Blocks`, and `mapDrift` compares only `Files`.
  A kept test whose `Blocks` is empty adds nothing to the covering union, so verdicts do not change.

## Supporting additions

- `gitrepo.Repo.FilesUnderDirAtRevision` and `fabricengine.Fabric.CodeTreeAt` list a directory's whole tracked subtree at a commit through go-git.
  These are the first changes outside `testcover` and `gatecli`.
  File content is still read only through `CodeFileAt` (PATTERN-fabric-boundary).
- `testcover.Toolchain.LinkedPackages` runs `go list -deps -test` under the slot's environment.
  It keeps the main module's packages and drops the external test package and the generated test main.
- `testcover.Toolchain.GoEnvironment` reads `go env GOVERSION GOOS GOARCH`.
- `testcover.FuncIndex` names the functions that hold a profile's covered blocks, parsing each file once.

## Design decisions

- Own files and linked files use the same ownership rule.
  So a Go file under a linked package's `testdata` counts as that package's, which matches D3's rule for own files.
  impactset's rule maps such a file to no package; using it here would only miss files, never add a false one.
- A dirty path is matched against the nested directories tracked at HEAD.
  So an untracked file in a new, untracked subpackage counts as dirty for the enclosing package.
  That can skip a package needlessly, but never misses a dirty file.
- A failing `go list` is recorded as a build failure for the package.
  A failing `go env` ends the walk, because no map can be stamped without it.
  `go env` runs at most once per call.

## Tests

- `testcover`: `TestCoveredFuncs` (tier 0); `TestLinkedPackages` and `TestGoEnvironment` against the stand-in `go`.
- `gitrepo`: a `FilesUnderDirAtRevision` row in `TestHistoryReads`.
- `gatecli`: `TestGateTestMapLinkedPackages` covers the record's D2 criteria on a fixture module with `caller`, `dirtier`, `linked` and nested `linked/inner`:
  - linked functions keyed by import path;
  - linked file hashes including a file in a subdirectory and leaving out the nested package;
  - own `testdata` hashes;
  - the out-of-process spawn verdict;
  - a test covering only linked code;
  - the format marker;
  - the coverpkg;
  - a linked-only landing leaving the map fresh;
  - a dirty linked file before the build and after the tests.
- `TestRedundancyStep`'s `beta` map now holds a `Linked` entry that drifted at the merge base and a covered linked function.
  The existing assertions on verdicts and drift stay unchanged.
- The testmap stand-in now also answers `go list` and `go env`, and records each coverage build's coverpkg.
