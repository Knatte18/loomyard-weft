```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke. I made no tool calls.

**Per behaviour: test that pins it**

| Behaviour | Pinned by |
|---|---|
| A linked worktree adds its git dir, then the common dir, after the records root | `TestPrepare_Launch` rows `RecordsTold`, `RecordsToldThroughLink` and `RecordsNotTold`. The `want` launch is compared with `reflect.DeepEqual` over `wantRoots`, which holds the order. |
| A plain clone adds one `.git` and `gitDirs` dedupes when the common dir equals the git dir | `TestPrepare_Launch` row `PlainClone`. A duplicated root fails the `DeepEqual`. |
| `OpenLaunch` carries the git dirs | `TestOpenLaunch` (`open_test.go:80`, `append([]string{records}, repo.wantGitDirs(anchor)...)`). |
| The guard rows' launch line carries the git dirs | `TestPrepare_InstallsGuardsFromItsInputs` (`writableRoots: repo.wantGitDirs(anchor)`). |
| Root order is records, then git dirs, then configured | `TestPrepare_Launch` row `ConfiguredRoots`. |
| A configured root that exists is kept and one that does not is dropped | `TestPrepare_Launch` row `ConfiguredRoots` (`cache` kept, `missing` dropped). |
| A relative `writable_roots` value refuses, names the key, carries "way forward", and writes neither `prompt.md` nor the lym home | `TestPrepare_RefusesBeforeWriting` row `RelativeWritableRoot`. |
| `writable_roots` is a guard key that a local override cannot set | `configreg_test.go:89`, `TestModules_EveryTemplateKeyClassified`, through `wantLocalDenied`. The engine's `ConfigSpec` feeds it. |
| The template names `~/.cache/go-build` | `TestLoadConfig_Template`. |
| No test reaches the real home's cache | The fixture's empty worktree-layer `writable_roots` (`fixture_test.go:60,73`). |
| The old assertion "writable-roots setting only when records told" was removed | Not a gap. The `DeepEqual` on `want.Cmd` replaces it, and `RecordsNotTold` still pins the case with no records root. |

**Findings**

1. **LOW, `design`.** Symlink resolution of the git dirs has no test.
   - Location: `internal/shuttleengine/codexengine/home.go:78-85`.
   - The summary and `doc.go` claim both git dirs are symlink-resolved.
   - `newRepoFixture` resolves its temp base with `EvalSymlinks` (`fixture_test.go:31`), and no row reaches the git dir or common dir through a link. Only the records checkout has such a row (`RecordsToldThroughLink`).
   - Dropping either `EvalSymlinks` call changes no expectation, so every test still passes.
   - A row where the gitfile's `gitdir:` target or the worktree path goes through a symlink would pin it. It can be added as a case to `TestPrepare_Launch`.

2. **LOW, `design`.** A `~/` value in `writable_roots` is untested.
   - Location: `internal/shuttleengine/codexengine/config.go:133-137`.
   - The rows use absolute paths. `TestLoadConfig_Template` pins only the raw `~/.cache/go-build` text.
   - A change in `configuredWritableRoots` that dropped the expansion would pass every test. That would include `EvalSymlinks(root)` on the unexpanded value, which skips the folder as missing.
   - The shipped default depends on that path.
   - `expandHomePath` has no `~/` test either. `t.Setenv` of `HOME` would keep a row out of the parallel set.
   - The test can name that reason in a comment, or put the home under an injected seam.

3. **NIT, `design`.** The empty `writable_roots` entry (`""`) has no row.
   - Location: `internal/shuttleengine/codexengine/prepare_test.go:374-380`.
   - The doc comment says "an empty or relative value" refuses.
   - The relative row covers `expandHomePath`'s shared branch, so this is only a case in the same row.

**Not findings**
- A permission error from `EvalSymlinks` on a configured root, a dangling symlink and a root that is a regular file have no row. The first is hard to build hermetically, and the others are minor variants of the missing-root path.
- A missing git dir or common dir in `readCheckout` is unreachable once `dotgit.FindRoot` has accepted the layout (its `IsRepository` check), so it needs no test.
- Test economy: the new rows are rows of existing tables, and no new test repeats another's coverage.

Tool calls: none; the manifest context held every file this lens needed.
