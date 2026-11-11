```yaml
lens: code-correctness
outcome: reported
```

Task restated: the code-correctness lens, one case per changed function or branch.

**Per-function list (case run, verdict)**
- `mapPackage`, first dirty check. Input: a linked package holds an untracked `notes.txt`. `dirtyOwnedFile` finds it, the package is skipped with the file's worktree path, and the earlier map is untouched. Correct.
- `mapPackage`, first-check skip and `--max`. The skip happens inside `mapPackage`, after `built++` at `testmap.go:404`. It therefore counts against `--max` and consumes cap, unlike the pre-walk and lock skips. The docs only state that after-test skips count. Wrong or inconsistent; see F1.
- `mapPackage`, second dirty check. Input: a test writes `linked/scratch.txt`. The status read after the last test finds it, no map is written, the package is skipped with the cause "... after its tests", and the earlier map stays. It runs before `changedFiles`. Correct.
- `mapPackage`, `LinkedPackages` failure. A non-cancel error becomes `writeFailed`, which carries `Format` and `OtherFiles`, with `Linked` nil. A cancellation returns the error. Correct.
- `mapPackage`, `ScanSpawns` failure. This writes a failed map, with the build failure recorded as "spawn scan: ...". It happens before the slot is taken and before the dirty checks, so a dirty tree does not stop it. Acceptable.
- `linked = Compact(Sorted(append(linked, importPath)))`. Empty `linked` gives just the package; a duplicate is compacted away. Correct.
- `linkedTrees`, import path equals the module path. It gives rel ".", and `path.Join(moduleRel, ".")` is the module root, so the whole module tree is read and nested directories are excluded. A path outside the module returns an error and the call ends. The description says a failing `go list` is a build failure, but this error ends the walk. It cannot happen under `Module.Main` filtering. Correct.
- `linkedHashes`, only the package itself. It returns nil, so `Linked` is omitted. Correct.
- `readPackageTree`, `nestedPackageDirs`, `ownsRelative`. Input: `linked/{linked.go, assets/data.txt, inner/inner.go}` gives nested `[inner]`, owns `assets/data.txt`, excludes `inner/inner.go`. Input: `testdata/x/y.go` is not buildable, so it is not nested and is owned. Input: a directory with only a `go.mod` is nested. Input: a directory holding only `.go` files that build tags exclude is treated as nested here. Impactset's `byDir` comes from `go list` and would not list it, so its files go to the enclosing package there. This over-excludes, never over-includes, and is a minor divergence; see F2.
- `packageTree.owns`. Input: `dir "."` calls `ownsRelative` directly. Otherwise `CutPrefix(file, dir+"/")` applies. A sibling `linkedx/f` is not matched. Correct.
- `dirtyOwnedFile`. An empty status gives `""`. With several dirty files, `slices.Min` picks the path-order first. An untracked file in a brand-new subpackage counts as the enclosing package's, which is conservative. Correct. Status paths are not verified to be per-file rather than collapsed directories; the fork did not read `Fabric.Status`.
- `changedFiles`. If HEAD equals `head` it returns nil at once. Otherwise it re-reads each tree at the new HEAD and lists added, removed and changed files. Own files are named relative to the package and linked files relative to the worktree. Correct.
- `fillMapTests`. Input: a run that covers only a linked function. `covered` is empty and `funcs` is non-empty, so the test enters the map with `Blocks` nil, which marshals as `"blocks": null`. Readers only range over `Blocks`. A run with neither is skipped. The function table is built from all runs and the indexes are rewritten, so index zero is not a stale placeholder. `m.Funcs` is nil when `runs` is empty or when `indexes` is empty. Correct.
- `FuncIndex.CoveredFuncs`. Input: a block with `Count` 0 is ignored; a nested package's block is dropped by `PackageBlocks`; a line past the file's end is an error, which `mapPackage` turns into `writeFailed`. A parsed file is cached. Correct.
- `writeFailed`. It clears `Blocks`, `Funcs` and `Tests` and keeps `Linked`, `OtherFiles` and `Files`. `writeMap` sets `Format` with the same `BuiltAt` as the map. Correct.
- `environmentStamp`. `go env` runs once per call and is cached. A failing `go env` returns an error from `writeMap`, which ends the call. A missing `go.sum` gives an empty hash. `Entries()` is sorted (`envconfig/config.go:97`), so `HubEnvDigest` is deterministic. Correct.
- `tiersDigest`. Tier numbers and env keys are sorted. Whether `r.tiers[n].Command` is pre-`{parallel}` is not shown in the diff. Not verified.
- `Toolchain.LinkedPackages`. Input: `m/q [m/p.test]` is cut to `m/q`. The external test package `m/p_test` with `ForTest` `m/p` is dropped. `m/p.test` with name `main` is dropped. Empty lines from non-module packages have one field and are skipped. A `main` package under test is kept. Correct.
- `Toolchain.GoEnvironment`. Three lines give the three values; any other count is an error. Correct.
- `gitrepo.FilesUnderDirAtRevision`. An invalid revision gives `ErrInvalidSHA`. An absent directory gives an empty result. Names are relative to the directory and sorted. Mode is filtered to regular files, so a symlink is skipped. Correct.
- Redundancy path, `redundancy.go:587-595`. A kept test with empty `Blocks` yields an empty `BlockSet`. Whether `JudgeAgainst` handles an empty kept run neutrally was not read; the summary claims it adds nothing to the covering union. Not verified.

**Findings**

- F1, LOW, `consistency`, `internal/gatecli/testmappackage.go:67-74` with `testmap.go:399-405`. The pre-build dirty skip fires after the package took a `--max` slot (`built++`). A package skipped for a dirty linked file therefore counts against `--max` and can leave later stale packages unbuilt. The existing pre-walk dirty skip does not count, and the Long help and `doc.go` do not say the new skip counts. Evidence: `testmap.go:399-405`, `testmappackage.go:79-86`.
- F2, LOW, `design`, `internal/gatecli/testmaptree.go:544-556`. `nestedPackageDirs` treats any buildable directory holding a `.go` file as a nested package. `packageOfPath` in `internal/impactset/derive.go:99-118` uses `go list`'s `byDir`. A directory whose Go files are all excluded by build constraints is nested here but is no package there. Its files then belong to no package in the map but to the enclosing package in impactset. The effect is only a missed hash, never a false one, but it departs from the record's "impactset's rule".

No other defects found in the cases above.

**Tool calls**
1. `grep` of `redundancy*.go` and `envconfig/config.go`: how redundancy consumes `Tests` and `Blocks`, and whether `Entries()` is sorted; neither was in context.
2. Read `redundancy.go:575-605`: the kept-test construction from map tests; the span was not in context.
