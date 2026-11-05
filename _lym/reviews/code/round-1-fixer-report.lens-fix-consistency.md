```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set (the `routeGitDirsThroughLink` helper, the `GitDirsThroughLink` row and the `TestPrepare_ExpandsHomeInWritableRoots` test) for disagreement with its surroundings, under the fix-consistency lens.

No findings.

Checked from my context:
- **`GitDirsThroughLink` row against `wantGitDirs`:** the row's `.git` is a gitfile, so `isDir(anchor/.git)` is false and the helper returns the resolved worktree git dir and common dir. Those match what `readCheckout` returns after `EvalSymlinks`.
- **Helper, `readCheckout` and `dotgit`:** `dotgit` does not resolve symlinks. The `commondir` `../..` joins lexically to `main-link/.git`, so `Base(CommonDir)==".git"` still holds. The main root resolves to `repo.mainRoot`, so trust roots and warnings stay unchanged.
- **Fixture header comment:** the `fixture_test.go` header still describes the fixtures, and `routeGitDirsThroughLink` fits it.
- **`TestPrepare_Launch` doc comment:** it describes the launch line and is not contradicted by the new row.
- **`~/cache` test:** it overwrites the `""` `writable_roots` layer that `addWorktree` wrote, uses a plain-scalar `~/cache` that is valid in a YAML flow list, and expects `cache` resolved under the already-resolved `repo.base`. The non-parallel comment matches the wording of `TestPrepare_AppendsStandingPassages`, and `HOME` and `USERPROFILE` cover `os.UserHomeDir` on every platform.
- **Twin tests:** `TestExpandHomePath` and `TestLoadConfig_Template` pin the same behaviors at a different layer and need no change.

No further tool calls were made; my context already held every changed file.
