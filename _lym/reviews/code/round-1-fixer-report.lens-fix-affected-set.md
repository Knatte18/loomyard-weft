```yaml
lens: fix-affected-set
outcome: reported
```

Checked set: the `EvalSymlinks` sites in `readCheckout` (F1) and the `~/` expansion sites (F2).

**F1** (git dir and common dir through a link): the new row `GitDirsThroughLink` fails if either the `GitDir` call (`home.go:78`) or the `CommonDir` call (`home.go:82`) is dropped. `dotgit` does no symlink resolution of its own, so the gitfile's `gitdir:` and the lexically joined `commondir` stay under `main-link`. The expected paths come from `wantGitDirs`, which are resolved.

**F2** (`~/` in `writable_roots`): `TestPrepare_ExpandsHomeInWritableRoots` covers the `writable_roots` site (`config.go:133`). The other `expandHomePath` callers are `home` and `operator_home` in `resolveHomes`, which `TestExpandHomePath` covers, and `skill_roots` in `skillload.go:114`. The review's premise was only `writable_roots`, so nothing is missing.

- **LOW**: the changed text is the `GitDirsThroughLink` row. It does not pin the `EvalSymlinks` on `filepath.Dir(geometry.CommonDir)` (`home.go:88`, the main root). In the row, dropping that call leaves `found.mainRoot` as the link path. The row only compares the launch command, which does not use `mainRoot`, so it still passes. Trust roots, which do use `mainRoot`, are not asserted in that row. F1 named only the git dir and common dir, so this is outside the finding. It is a sibling in the same function that the new row reaches but does not check. A trust-record assertion in that row would close it; I would not call it required.

Unchecked: none.

Tool calls:
1. `grep` for `EvalSymlinks` and `expandHomePath(` across `codexengine`. My context lacked a listing of every call site outside the files the fixer read (`codexengine.go`, `skillload.go`).
