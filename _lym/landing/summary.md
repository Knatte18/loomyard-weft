# testmap: record coverage across every linked in-module package

`lym gate testmap` now builds each package's coverage binary over every in-module package its test binary links, and the map records what each test covers there.
Before, a map held only the tested package's own blocks.
It could not tell which test of an importing package reaches a changed package, which a coverage-scoped gate needs.

## What changes

- The coverage build names every linked in-module package in its coverpkg, the package itself included.
  `testcover.Toolchain.LinkedPackages` lists them from `go list -deps -test`, leaving out the external test package and the generated test main.
- `testcover.Map` gains these fields:
  - `Funcs`: a table of linked packages' functions, each named by import path, file and function (`FuncKey`).
  - `MapTest.Funcs`: the indexes of the functions a test covers there.
  - `Linked`: the content hashes of each linked package's files, by import path.
  - `OtherFiles`: the hashes of the package's own non-Go files.
  - `MapTest.Spawn`: the `testcover.ScanSpawns` verdict of each test.
  - `Format`: a `MapFormat` marker holding the format version, the map's `Commit` and `BuiltAt`, and an environment stamp.
- A test that covers no block of its own package but a function elsewhere now enters the map.
- A package owns every tracked file under its directory down to the next package directory or nested module, `testdata` and `go:embed` targets in subdirectories included.
  `Fabric.CodeTreeAt` and `gitrepo.Repo.FilesUnderDirAtRevision` list a subtree at a commit; file content still comes through `Fabric.CodeFileAt`.
- The environment stamp holds the Go version and platform, the hashes of `go.mod` and `go.sum` at the commit, and digests of the mapped tiers' commands and env and of the hub's `env.yaml` variables.
  A failed-build map carries the marker too.
- Two fresh worktree status reads guard each package, each over every linked package's files:
  - Just before the build, a modified or untracked file skips the package, naming the file.
    The package does not count against `--max`.
  - After the last test, a dirty file or a changed hash at HEAD writes no map and leaves any earlier one as it was.
    The package counts against `--max`.
- The redundancy report keeps its own-package coverpkg and own-package blocks, so its verdicts do not change.
  `mapDrift` keeps comparing own-package files only.
- `docs/overview.md`, `pattern/PATTERN-test-speed.md`, and the `gatecli` and `testcover` package docs describe the new shape.

## Design decisions

- A linked package's coverage is stored as the functions a test covers, not as block keys with an import path.
  A function set stays valid when lines move inside a linked package, and the map stays small.
  This replaces the block-key form the Discussion's first record proposed.
- Staleness is unchanged.
  A change to a linked package's file alone leaves a map fresh, so a dependency-drift miss lasts until the map is rebuilt by the present rules: the package's own Go files or the age limit.
  The staleness policy belongs to the `testmap-upkeep` note.
- The first `Fabric.Status` read happens after the call holds its slot and lists the linked packages, so a package skipped on a linked file still costs a `go list`.
- Bound: an edit to a linked file made after the first status read and reverted before the second goes undetected.
  The map then holds coverage measured on the edited code under the committed hashes.
- Every existing map lacks the marker, so each package needs one rebuild before any selection can rely on the new fields.
  The present rules leave a map without the marker fresh until the age limit, or until a `testmap` call names the package.

## To check by hand

Not run:

- `lym gate testmap` against the real module, to see the build cost of a larger coverage binary per package.
- A package that links a widely imported package, such as `internal/fabricengine`: its map should hold `Linked` hashes and `Funcs` entries.
