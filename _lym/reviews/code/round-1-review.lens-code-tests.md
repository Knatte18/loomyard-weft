```yaml
lens: code-tests
outcome: reported
```

**Task: code-tests lens. Per behaviour, the test that pins it, or none.**

| Behaviour | Pinning test | Status |
|---|---|---|
| Hub-wide reconcile without `PruneUnknown` keeps an unknown key and reports `Preserved`, on a dry run and on apply, leaving the file unchanged and `Applied` false | `TestReconcileHubWideAt_PrunesUnknownKeysOnlyWhenAsked`, row 1 (`configsync_test.go` ~1392) | pinned |
| A retired key is still pruned beside a preserved key (the `withoutRetired` return path) | same test, row 2 | pinned |
| `PruneUnknown: true` prunes unknown and retired keys, and the file becomes the header-only base | same test, row 3 | pinned |
| `ReconcileAll` worktree file in a hub keeps an unknown key and reports `Preserved` without `PruneUnknown` | `TestReconcileAll_SparseLayers`, row "keeps a key the template lacks…" | pinned |
| `ReconcileAll` worktree file loses a retired key but keeps an unknown key | `TestReconcileAll_SparseLayers`, mixed row | pinned |
| `ReconcileAll` worktree file loses an unknown key under `PruneUnknown`, including the emptied-file delete | `TestReconcileAll_SparseLayers`, two `pruneUnknown: true` rows | pinned |
| Local file is pruned of unknown keys regardless of the option (`reconcileSparseFile` is called with `true`) | `TestReconcileAll_SparseLayers`, local row | pinned |
| The start-verb walk keeps an unknown board key (`fabric.yaml`) | `ensure_integration_test.go` `runStaleBuildReconcile` and `assertBoardStrippedOnce` | pinned for the board file only |
| The start-verb walk still heals a retired key, and the stamp and commit flow still holds | `seedRetiredHubWideKey` in the integration tests, and the `fabriccli` `staleHubConfig` fixture | pinned |
| The start-verb walk keeps an unknown key in a worktree file (`walk.go:285`, `ReconcileAll` without `PruneUnknown`) | none | **unpinned** |
| Clone (`clone.go:58`, `clone.go:122`) and `lym fabric reconcile` (`fabric.go:748`) keep unknown keys | none (`fabriccli` changed its fixture to a retired key, which only proves the heal) | **unpinned** |
| `lym config reconcile` in a hub (`configcli.go:560`) sets `PruneUnknown` and removes an unknown worktree key | none (`reconcile_integration_test.go:63` uses `stale_key`, but the run is standalone with no board dir, so it goes through the unchanged `reconcileBaseWorktreeFile`) | **unpinned** |
| `lym config reconcile --layer default --apply` (`hubwide.go:184`) removes an unknown key | none (`configcli_integration_test.go:509` seeds only retired keys, and those are pruned without the flag) | **unpinned** |
| `pruneSparse` when `yamlengine.Prune` reports a parent path against child paths, for nested keys, per-name maps and open maps | none (every row uses a top-level key) | **unpinned** |
| `logResult` adds `preserved` (`walk.go:379`) | none | unpinned, and low value |
| `lym config reconcile` help text | none | static prose, exempt |
| `DropRetired` export | existing config-load tests | not a new behaviour |

**Findings**

- **F-T1, MEDIUM, design.** `lym config reconcile` setting `PruneUnknown` has no test, in either the hub worktree-file path or the default-layer path.
  - Locations: `internal/configcli/configcli.go:560` and `internal/configcli/hubwide.go:184`.
  - Evidence: removing `PruneUnknown: true` from both fails no test. The only unknown-key reconcile test, `reconcile_integration_test.go:63`, is standalone. The default-layer integration tests at `configcli_integration_test.go:509` and `:337` seed retired or renamed keys only.
  - This is the Goal's "removal stays an explicit `lym config reconcile --layer default --apply`" half.
  - Suggested fix: add a row to the existing default-layer reconcile integration test with an unknown key, such as `newer_key`, in a board file. Assert that `--apply` removes it. If a hub-worktree reconcile fixture exists, add one worktree-file row too.
- **F-T2, LOW, design.** Nothing pins that the automatic callers leave `PruneUnknown` unset for worktree files. The callers are `hubreconcile/walk.go:285`, `fabriccli/clone.go:58` and `:122`, and `fabriccli/fabric.go:748`.
  - Evidence: adding `PruneUnknown: true` at any of these call sites fails no test. The walk's integration test checks only the board `fabric.yaml`.
  - Suggested fix: extend `runStaleBuildReconcile` with an unknown key in a pair worktree's sparse file, and assert it survives `Ensure`.
  - A `fabriccli` reconcile row would also cover the `fabric.go:748` call site. A clone row is optional.
- **F-T3, LOW, design.** `pruneSparse` is untested for nested unknown keys, per-name maps and open maps, where `Prune` reports a parent path on one pass and child paths on the other. All rows use top-level keys. The `slices.DeleteFunc` subtraction of `preserved` from `removed` relies on both passes reporting the same paths.
  - Location: `internal/configsync/configsync.go:652`.
  - Suggested fix: add a row with a nested unknown key beside a nested retired key. Use a module that has a nested retired key in its spec. Assert `Removed` and `Preserved` are disjoint and the file bytes are correct.

**Tool calls**
1. `grep` for the `logResult` callers plus unknown-key fixtures in the `configcli`, `fabriccli` and `hubreconcile` tests. My context lacked the `logResult` call condition (results are logged only when `Applied`) and the existing fixture keys.
2. `grep` for `reconcile --layer default` tests, plus a read of `reconcile_integration_test.go`. My context lacked which fixtures the `lym config reconcile` tests use and whether they run standalone or in a hub.
