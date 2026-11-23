```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens report on the decision record (D1–D9) for config-keys-nested-by-area.

**Findings**

1. **BLOCKING** · D8 (key builders) and D7
   - **Finding:** `lym beam merge-parent` builds a run_settings object whose keys come from `reviewbudget.SoftKey` and `reviewbudget.HardKey`. D8 makes those builders return dotted paths, so the filed task carries `{"loom": {"code.max_bounces": N, "code.hard_max_bounces": M}}`, a literal dotted key in the module object. The record is silent about this.
   - **Evidence:** `internal/beamcli/mergeparent.go` around lines 81–85 does `"loom": map[string]any{reviewbudget.SoftKey(perch): rounds, reviewbudget.HardKey(perch): ...}`. That map goes straight to `deps.FileTask`, then `configreg.PrepareRunSettings` and `yaml.Marshal`.
   - **Effect:** a key named `"code.max_bounces"` is not the nested `code: {max_bounces: N}`. Either `ValidateLocal` refuses it and merge-parent breaks, or it gets through and the loom loader ignores it, so the merge task silently runs on default budgets. Any other place that uses a builder's result as a map key, rather than as a path for `configengine.Set` or lookup, breaks the same way.
   - **Fix:** D8 needs to say how a builder's dotted path becomes a nested object for run_settings. That could be a configreg or yamlengine helper that nests dotted keys, or a run_settings shape decision that accepts dotted keys inside a module object.

2. **MEDIUM** · D7, where the board carry runs
   - **Finding:** D7 says "the hub-wide walk … through `boardengine`'s own write path under its lock". The hub-wide walk runs inside a `fabricengine.BoltWrite` closure in `reconcileHubWide`. `boardengine` writes through `BoltWrite` too, and both take the same board write lock.
   - **Evidence:** `internal/hubreconcile/walk.go` `reconcileHubWide` runs `configsync`, then `applyReinterpretations`, then `afterHubWideWrite`, all inside `write.Write`. `internal/boardengine/board.go` uses `BoltWrite`. `fabricengine`'s Bolt tests take `lock.TryAcquireWriteLock` on the board write lock.
   - **Effect:** if D7 sits beside the reinterpretations, as "the hub-wide walk … behind a board marker" suggests, it nests a board write inside a board write, which deadlocks or double-commits. If it sits outside, the marker and the board.json rewrite land in separate commits.
   - **Fix:** the record should place D7 outside the hub-wide `BoltWrite`, say the marker is written in or after the board write, and say the order. Concurrent board pushes are already a live problem on the board (entry `weft-main-push-race`).

3. **LOW** · D7 and Open risks
   - **Finding:** after D7's once-per-hub marker, a run_settings write from a machine on an older build still uses old names. `boardengine`'s field replay applies it, so old names reappear on the board and are never carried again. Seed-Child then refuses that entry under D4.
   - **Evidence:** `internal/boardengine/replay.go` line 208 replays `run_settings` field by field.
   - **Fix:** the mixed-build open risk names only the hub base `loom.yaml`. It should also name board run_settings and `profiles.yaml`, which have the same once-only marker problem.

**Paths traced, per decision**

- **D7**
  - Seed-Child `writeRunSettings` (`internal/battenshed/seamchild.go`) reads the board afresh each call, after `hubreconcile.Ensure` → survives.
  - The board upsert's run_settings check (`internal/boardcli/runsettings.go` `prepareRunSettings`, then `PrepareRunSettings`) refuses old names → survives (that is D4).
  - The board's run lock (`internal/boardengine/runlock.go`) never compares RunSettings, so D7 can rewrite a running entry → survives.
  - Field replay → breaks under a mixed build (finding 3).
- **D4, D6**
  - `lym config profiles --set` runs `CheckLoad`, then `checkProfiles`, then `PrepareRunSettings`, and refuses old names → survives.
  - The walk calls no `CheckLoad`, so a tuned flat profile is not refused before D6 runs → survives.
- **D5**
  - `reconcilePair`, the pair call, runs `configsync` → survives.
  - `lym config reconcile` with PruneUnknown: the move comes before the prune → survives as the record says.
  - A mid-merge worktree is skipped, which keeps the stamp stale so it is retried → survives.
  - An incomplete pair is skipped without making the stamp stale. Seed-Child rewrites its local layer from the carried run_settings, and its tracked worktree file comes from a parent that was already moved → survives.
- **D6:** the profiles reinterpretation runs after the `configsync` merge in the same closure → survives.
- **D8**
  - `FlagCodeAutoAccept` runs after `configsync` in the same closure → survives with the new path read.
  - `loomcli/circling.go` compares `caps.HardKey` with `HardKey(plan)`, both from the same builder → survives.
  - `beamcli` merge-parent uses builder results as map keys → breaks (finding 1).
- **Constraints:** `configengine.TopLevelValue`'s only non-test caller is `dobbyengine/tasksetting.go`, which reads beam → the claim holds.

**Tool calls**

1. Grepped `TopLevelValue(` callers and the run-settings code in battenshed, battencli and boardcli, to check the record's one-caller claim and find the run-settings write and read paths.
2. Read `internal/boardcli/runsettings.go` (lines 28–80) and grepped boardengine for run_settings, to see the board-write validation, the run-lock comparison and the replay path.
3. Grepped `CheckLoad` callers and read `reconcileHubWide` in `walk.go`, to learn whether the walk validates profiles and which write closure and lock the hub-wide step runs under.
4. Grepped for persisted key-name fields and for `SoftKey(` / `HardKey(` callers outside reviewbudget, and for `BoltWrite` in boardengine, to find readers of the key builders and the board's write path.
5. Read `internal/beamcli/mergeparent.go` (lines 70–100) and grepped the fabric Bolt lock, to confirm the builders feed a run_settings map and that both writes share one lock.
