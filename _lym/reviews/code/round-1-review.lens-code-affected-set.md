```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens, covering what the diff's changes force outside the lines they touch.

**Per change, the set found:**
- `SpawnVerdict` now has JSON tags (`out_of_process`, `unresolved`, both omitempty).
  - Consumers: `testcover/verdict.go` (`Run.Spawn`), `gatecli/redundancy.go:242`, `testcover/spawnscan.go` and the tests.
  - They read the struct in process and never serialize it except through `MapTest.Spawn`, so nothing else changes shape. Covered.
- New `Map` fields (`OtherFiles`, `Linked`, `Funcs`, `Format`) and `MapTest` fields (`Funcs`, `Spawn`).
  - Readers: `redundancy.go` reads `Blocks`, `Files`, `BuildFailure`, `Commit` and `BuiltAt` only, and `checkMapBlocks` covers `Blocks` only.
  - `stale()` in `testmap.go:463` compares `Files`, age and failure only.
  - All are `omitempty` or additive, so older maps and undeclared keys still read tolerantly (PATTERN-tolerant-records). Covered.
  - The redundancy test fixture sets `Linked`, `Funcs` and `Tests[0].Funcs` on `beta`'s map and keeps its verdict and drift assertions. Covered.
- `Fabric.CodeTreeAt` and `Repo.FilesUnderDirAtRevision`.
  - Neither is added to the interfaces that list `FilesInDirAtRevision` (`reportengine/source.go:26`, `tools/tokencount/calibrate.go:42`), and nothing needs them there.
  - File content is still read only through `CodeFileAt`, so PATTERN-fabric-boundary holds, and `gitrepo` uses `readGoGit`, so PATTERN-gitrepo-client-boundary holds. Covered.
- `BuildBinary` coverpkg, new status reads and the `mapPackage` flow.
  - Docs: `docs/overview.md`, the `internal/gatecli/doc.go` testmap section, the `testMapCmd` help text in `testmap.go` and `PATTERN-test-speed.md` all describe the new coverpkg, the two reads and the map contents.
  - `PATTERN-test-speed.md` still says a map is rebuilt on own-Go-file change, age or failure, which matches the unchanged `stale()`. Covered.
  - `docs/overview.md` and the `doc.go` text still say the pre-walk check is per own Go file, and that is accurate. Covered.
- `redundancy.go`'s own-package build (`BuildBinary(importPath, importPath, …)` at line 473) and `mapDrift` over `m.Files` are untouched. Covered.
- `internal/testcover/doc.go` describes the new map shape, `FuncIndex`, `LinkedPackages` and `GoEnvironment`. Covered.
- Declared PATTERN changes: the constraints section declares none, so nothing is owed.
- Commit messages: all three feature commits match the diff and the amended D13 text (function sets, not import-path block keys). They carry no `Pattern-Departure:` trailer and need none.
- Stencils: none changed.

**Findings:** none.

**Tool calls:**
1. Grepped for `FilesInDirAtRevision` and `CodeDirAt`, printed `testcover/verdict.go`, and grepped for `OutOfProcess` and `out_of_process`. Context lacked the other consumers of the sibling reader and whether the verdict type is serialized.
2. Grepped the md, yaml and tmpl files, and `docs`, `pattern` and the two `doc.go` files, for text describing the coverage map. Context lacked the docs outside the diff that might describe the map.
