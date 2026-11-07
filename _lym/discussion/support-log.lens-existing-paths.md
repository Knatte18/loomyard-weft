```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: tracing the paths that already write, reset or read the state each decision changes, and whether each decision survives them.

**Findings**

1. `MEDIUM`, D1: the record never says the main worktree's path must be fully resolved through symlinks, as git resolves it.
   - Evidence: on the scratch repository, the record's own exploration ran `git -C .scratch/wt/main` through a symlink (`.scratch` points into `fabric-test-speed-weft/.lym/scratch`). The `worktree` line for main came back fully resolved.
   - Git writes a linked worktree's path into its `gitdir` file when the worktree is added. A reader that builds the main path from the common dir, as reached from the caller's path, keeps any symlink in that path.
   - `PrimeName`, and through it `IsPrimeWorktree` and `RecordsRepoRoot`, read only `filepath.Base` of the main path, so they survive. Callers that compare whole `entry.Path` values against paths from `lymcwd` would misread the state, unless both sides are resolved the same way. Those callers are `prune`, `cleanup`, `addadopt` and `cleanupremotewarp` in the List caller list, plus `websterengine`'s canonicalising filter.
   - D1's parity criterion runs on a temp directory with no symlink, so it cannot catch this.
   - Fix: D1 states that the main path is resolved through symlinks as git resolves it, and a D1 acceptance bullet runs the parity check through a symlinked path.

2. `MEDIUM`, D2: "the base shape's build plus the build function" does not say where the base build happens. Building it through `CopyHub` would break the template.
   - Evidence, template lifetime: `CopyHub` takes `tb.TempDir()`, which is removed when the first caller's test ends.
   - Evidence, the guard: `hubAt` calls `RegisterTeardown(tb, …)`, which is a `tb.Cleanup`. D2's builder TB refuses `Cleanup`, so a base built as a copy fails its own build.
   - The shape builds that work today run `cloneHub` and `AddPair` into a `lymtemp` root and register no cleanup (`internal/hubforge/hub.go`: `RegisterTeardown` is called only from `NewHub` and `hubAt`).
   - Fix: D2 states that a registered template is built in its own `lymtemp` root, the way `templateFor` builds a shape: bares, `cloneHub`, the shape's pairs, then the build function. It is never built from a copy.

3. `LOW`, D2 and D3: `hubAt` and `templateFor` take a `Shape` and fail anything outside `Shapes()`, so the copy of a registered template needs its own lookup.
   - Evidence: the closed-set check in `templateFor`, and `hubAt` re-entering it.
   - The record says the copy goes "through the same relocation", which holds for `relocateHub`. `hubAt`'s rebase also reads the template's own `Hub`, so a registered template must keep a `Hub` that names the pairs its builder added.
   - This survives once the lookup is keyed; it is a note, not a break.

4. `LOW`, D3: if the packing step is `git gc`, it also runs `worktree prune`, `reflog expire` and `rerere gc` on the template's repositories.
   - The template's linked worktrees exist, so nothing is pruned today, and the result survives.
   - `git repack -ad` plus `git pack-refs --all` is the narrower step that does only what D3 states.
   - Relocation's text rewrite and its "nothing names the old root" check skip binary packs, and `packed-refs` holds no path, so the claim survives.

5. `LOW`, D5: `InstallPostCheckoutHook` returns early when the existing hook holds the sentinel, so hubs already installed keep the old script that calls `dirname` and `basename`.
   - Tests build fresh hubs, so the test saving holds, and the old script still works.
   - The record could say that existing hubs keep the old hook. Nothing breaks.
   - `internal/fabricengine/hook_test.go` refers to `dirname` or `SCRIPT_DIR` text. It is a reader that has to change with D5, not a path that undoes D5.

**Paths traced, per decision**

- D1:
  - `PrimeName`, `IsPrimeWorktree` and `RecordsRepoRoot`: survive, since they read only the base name.
  - Full-path comparers among the List callers, and `websterengine` `otherWorktrees`: survive only if the main path is resolved as git resolves it (finding 1).
  - git itself writing `.git/worktrees` during add, lock and prune: the reader sees the current files, so this survives. Parity with porcelain's prunable mark was confirmed in the scratch repository.
- D2:
  - `templateFor`'s closed-set check, and `hubAt` re-entering it: needs a keyed path (finding 3).
  - `cloneHub` and `AddPair`/`AddPairWith`: register no cleanup, so they survive under the builder-TB guard.
  - `CopyHub`/`hubAt` as a base: breaks (finding 2).
  - The ref-counted `markFixtureBuild`: survives concurrent builds.
- D3:
  - `relocateHub` rewrite and verify after packing: survive.
  - The `SharedHub` cleanup fingerprint, which stats every regular file: survives.
  - The `poisoned` check on `CopyHub`: survives.
  - A pair branch based on the board's pre-amend commit is the same as today, and survives.
- D4: `git init`'s own config write comes before the Go write, and a second `[core]` section is valid, so it survives.
- D5:
  - `InstallPostCheckoutHook`'s sentinel early return: existing hubs keep the old script (finding 5).
  - `chainUserHook` concatenating the wrapper: survives with the `.` fallback.
- D6: the refusal rows of `TestAnchorChange` on `SharedHub`.
  - In `ChangeAnchor`, `validateAnchorTarget` and `ValidateAnchorRel` run before `ensureWeftLockDirAt` and the move lock, so a refusal writes no lock file. The fingerprint survives for those rows.
  - Any logging the verb's entry point does into the hub was not traced.
- D7: `newBadReedUpFixture` runs `SeedConfig` with the corrupted reed line and `AddPair("loom-smoke-task")` before the per-site seed. The template base is the record's claimed base, so it survives.

**Tool calls**

1. Grepped `Cleanup`, `Setenv`, `RegisterTeardown`, `AddPair` and `cloneHub` in hubforge, to learn whether the build helpers register cleanups. The context had only the template file's top part.
2. Read the install logic in `hook.go` and grepped the tests that cite `dirname`, to see whether an existing hook is rewritten. The context had only the preamble const.
3. Read `newBadReedUpFixture`, to check the order of seed and pair against D7's template base. The context had only the subagent's summary.
4. Read `fingerprint` and `PrimeName`/`IsPrimeWorktree`, and the `SharedHub` sites, to see what the walk covers and how List's main path is consumed. Not in context.
5. Read the start of `ChangeAnchor`, to check whether D6's refusal rows write before refusing. Not in context.
