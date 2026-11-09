# internal/testcover engine, with testtiming's package-scoped verdict

`go run ./cmd/testtiming -redundancy` judged each test over every block of the module.
Blocks in fixture packages left almost every test unique,
so the report listed no candidate for a package like `internal/fabricengine`.
The runner and verdict also sat in a `main` package, which the coming map verb and gate step cannot import.

## What changes

**New package `internal/testcover`.**
It holds what `cmd/testtiming` had, now exported:

- `Toolchain.BuildBinary`, `ListTests` and `RunAlone` build a package's coverage binary, list its top-level tests and run each test alone with its own profile.
  The go command and environment are fields, so tests can swap in a stand-in.
  Every spawn is logged through `internal/logger`, since the package is reachable from future `lym` verbs.
- `ParseProfile` returns structured `ProfileBlock`s, covered or not.
  `PackageBlocks` keeps the blocks in the package's own files, and `CoveredPositions` builds the block set a verdict compares.
- `Judge` is the old `classify`: the contained set, the greedy removable set (slowest first) and the covering tests.
  The eligibility rules have not changed: a skipped, failed, coverage-free, out-of-process or unresolved test is not judged.
- `ScanSpawns` and `ScanKeeps` are the spawn scan and the keep scan, moved from `spawnscan.go` and `keep.go` without changes.
- `KeyBlocks` keys a block by content (D3): its package-relative file, its enclosing function (`Type.Method` for a method, `PackageLevelFunc` outside any function), the SHA-256 of its source text, and its ordinal among blocks equal in those three.
- `Map`, `MapTest`, `MapPath`, `ReadMap` and `WriteMap` define the per-package coverage map record (D4).
  `WriteMap` writes it atomically through `internal/state`, which keeps unknown keys, with its lock path from `lock.TrackedFileLockPath`.

**`internal/lymdirs`** adds `TestCoverDirName`, the maps directory under the hub's board-level `.lym`.
The map verb composes the full path from it.

**`cmd/testtiming -redundancy`** calls the engine (D2).
It keeps one coverage binary per package, still instrumented over the whole module, and judges each test twice from the same profiles:

- once over the package's own blocks, which decides the candidates and the removable set;
- once over every block, which fills a new `Contained module-wide` column.

Flags, `-pkg` splicing, keep handling and report writing work as before.

**`cmd/lym/tierpurity_test.go`**'s allowlist entry for the spawn scan's fixture file now points to `internal/testcover/spawnscan_test.go`.
`spawnobservability_test.go`'s entries for `cmd/testtiming` stay, because `redundancy.go` still runs `go list` and `go test -json`.

## Design decisions

- The tool compares blocks by profile position, not by content key.
  Within one run, position and content identify the same block, and positions need no source parse.
  Content keys are for the map, which compares blocks across commits.
- `KeyBlocks` computes ordinals among the blocks it is given, so a caller passes every block of the package's profile, covered or not.
  It keys from a profile, which supplies the block boundaries, and then reads only the source files.
- Under the package view, a test that covers only other packages' blocks is listed as `no blocks`, because it covers nothing of its own package.
- The map stores tests as a slice sorted by the producer.
  `internal/state` merges a slice element's unknown keys only when the element is unchanged.

## Tests

- `internal/testcover`:
  - `TestJudge` (moved from `TestClassify`, now with a failed-test row);
  - `TestParseProfile`;
  - `TestKeyBlocks`: a shifted block keeps its key, an edited block gets a new one, and same-named methods on two receivers get different keys;
  - `TestWriteMapReadMap`: the map round-trips, and an unknown key is read without refusal and kept on rewrite;
  - the moved `TestScanSpawns` and `TestScanKeeps`;
  - `TestBuildBinaryAndListTests` and `TestRunAlone` (tier1), which drive the runner against the test binary re-executed as a stand-in `go` and a stand-in coverage binary, so no test runs `go test`.
- `cmd/testtiming`: `TestJudgePackage` gives a candidate whose only unique block lies in another package, and checks it is listed as a candidate and marked not contained module-wide.
  `TestRenderPackage` covers the new column.

## Not changed

`docs/benchmarks/test-redundancy.md` is not regenerated.
A `-pkg` run splices the new four-column table into the measured sections and leaves the old three-column sections of other packages as they are until those packages are measured again.
