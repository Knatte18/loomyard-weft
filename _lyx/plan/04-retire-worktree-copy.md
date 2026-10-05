# Card 4 — `lyx config reconcile` skips hub-wide modules and retires their per-worktree copies

**Edit:**
- `internal/configsync/configsync.go`
- `internal/configsync/configsync_test.go`
- `internal/configcli/configcli.go`
- `internal/configcli/reconcile_integration_test.go`
- `internal/configcli/configcli_integration_test.go`
- `internal/fabriccli/clone.go`
- `internal/fabriccli/cloneconfigcommit_integration_test.go`
- `internal/fabriccli/cli_test.go`
- `internal/hubforge/hub_test.go`

**Uses:**
- `plan:internal/configsync#ReconcileHubWideAt`
- `internal/configreg#Module`
- `internal/fabricengine#BoardDir`
- `internal/fabricengine#CommitAnchoredPaths`

**Intent:**
Change `ReconcileAll` to `ReconcileAll(baseDir, boardDir string, apply bool)` (D6), and replace its `m.Name == "fabric"` skip with a branch on `m.HubWide`, so no site decides hub-wide handling by module name.
A hub-wide module is never reconciled or written in the worktree.
Its result is decided by the two files:
- No per-worktree copy at `configengine.ConfigFile(baseDir, name)`: an empty result.
- A copy, and no hub file at `configengine.ConfigFile(boardDir, name)`: the copy is left in place, never deleted.
- A copy and a hub file, where the copy adds nothing to the hub file: the copy is removable.
  Every leaf of the copy's decoded YAML is present in the hub file at the same key path with an equal value, and lists are compared whole.
  The result reports the copy as removable, and `--apply` deletes it.
- A copy and a hub file, where the copy holds a key path the hub file lacks or a value that differs: the copy is divergent and is left in place.
  The result lists each such entry as `<dotted.key>: <value>`, sorted.

`Result` gains `Retired bool` (the copy is removable; `Applied` reports the deletion happened) and `Divergent []string`.
Bound: the only file this deletes is `_lyx/config/<module>.yaml` of a registry-marked hub-wide module, only under `apply`, only with the hub copy present and holding everything the per-worktree copy holds, so no unmerged vocabulary edit is dropped.

`lyx config reconcile` (`runReconcile` in `configcli.go`) passes `fabricengine.BoardDir(l.HubPath)`.
It adds `retired` and `divergent` to a module's JSON object only when set, as `migratedFrom` is handled today.
Update the reconcile subcommand's `Long` text to say hub-wide modules are reported, not reconciled.
`CloneAndWire` passes `res.BoardDir`.
Its weft commit of applied per-worktree paths now also carries a retired copy's deletion, since a retired result is `Applied`.

Every test that used `board` as the generic per-worktree module moves to another per-worktree module with a non-seed-only template, such as `loom`:
- `TestReconcile_Apply` and its dry-run sibling in `reconcile_integration_test.go`;
- the set-then-reconcile test in `configcli_integration_test.go`;
- the per-worktree config assertion in `fabriccli/cli_test.go`'s clone end-to-end test;
- `TestNewHub_ConfigMaterializedWithoutSeeding` and `TestSeedConfig_VisibleFromWarpSide` in `hubforge/hub_test.go`;
- `cloneconfigcommit_integration_test.go`.
Remove `TestReconcileAll_BoardOpenMapsCarriedWhole`: no per-worktree module declares open maps any more, and card 3's `ReconcileHubWideAt` test pins `board`'s open maps being carried whole.
Update the comments in those tests that explain `fabric` being skipped by name.

Coverage: one table-driven test in `configsync_test.go` over a hub-wide module (`board`) with rows:
- no per-worktree copy → no `Retired`, no `Divergent`, no file written;
- copy and no hub file → copy kept;
- copy subsumed by the hub file → dry run reports `Retired` and keeps the file, apply deletes it;
- copy holding a label the hub file lacks → `Divergent` names `labels.<name>: <description>` and the file is kept.
Every row also asserts that the per-worktree `board.yaml` is never created and `fabric.yaml` is never written.
In `reconcile_integration_test.go`, add one assertion that `--apply` over a subsumed per-worktree `board.yaml`, with a hub file present at the temp repo's board dir, reports `retired` in the JSON and deletes the file.

**ImpactSummary:** `ReconcileAll` gains a `boardDir` parameter, with its two production callers and their tests updated; clone stops materializing a per-worktree `board.yaml`, and tests that used `board` as a generic module move to `loom`.

**Commit:** 4: configsync: reconcile retires per-worktree copies of hub-wide modules
