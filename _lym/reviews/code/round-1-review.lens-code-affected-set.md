```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change, what else it touches and whether that is covered.

**Affected set, per change**

1. **`PruneUnknown` on `Options` and `HubWideOptions`.**
   - Set to true only in `configcli/configcli.go:560` and `hubwide.go:184`. Covered.
   - Left unset, so they now keep unknown keys: `hubreconcile/walk.go:211` and `:285`, `fabriccli/clone.go:58` and `:122`, `fabriccli/fabric.go:748`, and `fabriccli/cleanhub_integration_test.go:137`. Covered.
   - Tests: the `ReconcileAll` and `ReconcileHubWideAt` cases in `configsync_test.go`, `ensure_integration_test.go`, and the `fabriccli` stale-config fixture. Covered.
2. **`Result.Preserved`.**
   - Logged in `hubreconcile/walk.go:379`. Covered.
   - Not surfaced by the `fabriccli` clone and reconcile callers. See F1.
3. **`DropRetired` exported (`configengine/layers.go`, `config.go:230`).** Covered; the one internal caller is updated.
4. **`PATTERN.md` and `pattern/PATTERN-config-layers.md` text, plus the `hubreconcile/doc.go` section.** Present and consistent with each other. One stale line remains in the same background file. See F3.
5. **Doc comments.** `configsync.go` and `fabriccli/fabric.go:726` are updated. `configcli/doc.go:84` ("keys the template lacks are pruned") describes `lym config reconcile`, which still prunes, so it is correct.
6. **Declared PATTERN changes.** My context lists no declarations, so there is nothing to check.
7. **Commit message.** Matches the diff. No `Pattern-Departure` trailer is needed.

**Findings**

- **F1, LOW, consistency.** `fabriccli/clone.go:58` and `:122`, and `fabric.go:748`.
  - The PATTERN text says every other reconcile "keeps such a key and reports it as preserved".
  - Only the `hubreconcile` walk reports it (`walk.go:379`).
  - `lym fabric reconcile` and clone discard `Preserved`.
  - Either surface it in their output or narrow the wording to "logs it in the walk".
  - Evidence: the `clone.go` loops read only `Applied`, `Layer` and `Module`; the `fabric.go` write closure reads only `Applied`, `Module` and `MigratedFrom`.
- **F2, LOW, scope.** `configsync.go:503`, `reconcileBaseWorktreeFile`.
  - The standalone worktree base (`boardDir == ""`) still removes every key the template lacks through `yamlengine.Reconcile` (`configsync.go:546`). `PruneUnknown` does not gate it.
  - That contradicts the new PATTERN sentence "only `lym config reconcile` removes a base or worktree key the template lacks".
  - The `ReconcileAll` doc says the standalone worktree file is "fully reconciled" but does not name this exception.
  - I did not trace whether an automatic caller can reach this path. The clone and walk callers pass a non-empty board dir.
  - Narrow the PATTERN and `doc.go` wording to hub files, or say the standalone base is out of scope.
- **F3, LOW, consistency.** `pattern/PATTERN-config-layers.md`, "Local files" section.
  - It says reconcile deletes "a local file whose retired keys it pruned to nothing".
  - Local files are now pruned of every unknown key (`configsync.go:452`, `pruneUnknown` true).
  - The new "Keys the template lacks" section states this correctly, so the older line is now narrower than the code.

**Tool calls (2)**
1. Grepped for "template lacks" and "prunes keys" prose across `*.go`, `*.md` and `*.yaml`, to find docs the diff left stale. My context did not hold these.
2. Read `fabriccli/clone.go:40-75` and `:115-140`, and `fabric.go:740-780`, to see whether `Preserved` is surfaced. The context held only the diff hunks.
