```yaml
lens: existing-paths
outcome: reported
```

Task: the existing-paths lens. I traced the code that already writes, resets or reads the state each decision changes.

**Paths traced, by decision**

- **D1, `List` callers** (`OpenParent` via `matchParentBranch`, `PrimeName`, the Remove and Destroy ownership gates, hubpull's worktree-list order): **survives**, as long as D1 matches git's porcelain output. Each caller reads only path, branch, Main and Prunable, and D1 keeps that result.
- **D1, `websterengine.otherWorktrees`** (`gitwrap.go:322-347`): **survives**. It uses paths alone, after `canonicalPath`. From a linked worktree it reaches the common dir through `dotgit.Read`.
- **D1, hubforge relocation of `worktrees/&lt;id&gt;/gitdir` and the `.git` files**: **survives**. Absolute gitdirs are text that relocation rewrites; relative ones need no rewrite.
- **D2, `markFixtureBuild` under concurrent builds** (`hub.go:41-65`): **survives**. The mark is ref-counted and goroutine-safe.
- **D2, the base build path `cloneHub` → `forgetCloneCorrespondence` → `AddPair`** (`hub.go:289-362`, `414`): **survives** D2's `TB` refusal. The only `Cleanup` is in `RegisterTeardown` (`hub.go:381`). The only `TempDir` calls are in `copyBares`, `NewHub` and `CloneSecond` (`hub.go:156`, `258`, `280`), none of which the shape build calls. The amend in `forgetCloneCorrespondence` leaves the old commit reachable only from the reflog, and `repack -a` packs that, so it does not stay loose.
- **D3, `relocateHub` and `findRootReference` over packs and `packed-refs`** (`template.go:255-389`): **survives**. Pack files contain NUL bytes and are never rewritten, `packed-refs` holds no paths, and reflog text is still rewritten as today.
- **D3, `commitRelocatedBinding` against packed refs**: **survives**. The push turns the packed tracking ref back into a loose ref and writes the reflog entry `trackingReflog` reads (`reflog.go:32-40`).
- **D3, existing tests that read loose objects or ref files**:
  - `housekeeping_integration_test.go:60-79` and `:144-158` create their own loose objects on a `CopyHub`: **survives**.
  - `cleanupremove_integration_test.go:342` writes a lock under `refs/heads`. That test runs on a `CopyHub` (`:96`), and `pack-refs` never removes the `refs/heads` directory: **survives**.
  - `push_destination_integration_test.go:111` and `warpbinding_clone_integration_test.go:446` use their own repositories, not hubforge templates: **unaffected**.
- **D3, `SharedHub`'s entry check and cleanup walk**: **survives**, unchanged. The only `SharedHub` site outside hubforge (`warplayout_test.go:26`) only reads.
- **D4, `rewriteOriginURLInConfig`** (`gitkit.go:220-240`), which reads `.git/config` and expects exactly one `url` under `[remote "origin"]`: **survives**, because D4 writes no remote keys.
- **D4, `initBareRemote` running `remote add` after the init write**: **survives**, because the calls run one after another.
- **D4, the bare initialisers** `gitkit.initBareRemote` (`gitkit.go:149-159`) and `hubforge.initBareRepo` (`hub.go:469-479`): **breaks** the record's claim. See finding 1.
- **D5, the installer's sentinel rule**: **survives**. The record states that installed hubs keep the old script. Templates are built fresh in each binary, so they get the new script.
- **D5, the chain wrapper's `$0`**: **survives**. Git calls the hook through a path that contains a slash, and the record adds a `.` fallback for one without.

**Findings**

1. MEDIUM, consistency. Location: D4 and the D4 acceptance criterion. D4 says every `gitkit` and `hubforge` initialiser writes five keys, including `user.name` and `user.email`, "as today". The bare initialisers write only `core.fsmonitor`, `maintenance.auto` and `gc.auto` today.
   - So the criterion "a repository from each initialiser reads the five keys at today's values" either fails for the bare repositories, or makes the plan add `user.*` to bares, which contradicts "keys and values stay as today".
   - Evidence: `internal/gitkit/gitkit.go:149-159` (`initBareRemote`) and `internal/hubforge/hub.go:469-479` (`initBareRepo`) write three keys; `gitkit.go:131-138` and `hub.go:457-466` write five.

No other traced path undoes or misreads a decision.

Tool calls:
1. hubforge `hub.go`, `RegisterTeardown` and `gitkit.Git`/config: to check whether build helpers call `Cleanup`, `TempDir` or `Setenv`, and what else reads `.git/config`. My context had only grep hits.
2. A grep for readers of loose objects and ref files, the `TestStoreHousekeeping` fixture and `reflog.go`: to find existing code D3's packing could break. Not in the manifest.
3. The loose-object span of `warpbinding_clone_integration_test.go` and the housekeeping fill/assert spans: to check whether those tests depend on a template's loose objects.
4. `cloneHub`, `forgetCloneCorrespondence` and the `FixtureGitEnv` readers: the base build path a D2 template runs and its process-global env mark. Not in the manifest.
5. The ref-lock spans of `push_destination_integration_test.go` and `cleanupremove_integration_test.go` and their fixtures: to check whether the `pack-refs --all` from D3 removes the directories those lock files go into.
