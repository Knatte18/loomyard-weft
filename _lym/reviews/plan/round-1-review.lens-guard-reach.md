```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch or test row the cards add, I traced whether a real entry point can reach it and whether the guarded site recurs on a path no card guards.

**Per guard: reachable or not**

- **Card 2, `ValidateLocal` moved-key check** (local file `old_key`, scalar `area`, mapping `area` admitted): reachable. `stackLayers` calls `ValidateLocal` on every local layer at load (`internal/configengine/layers.go:476`). Through `PrepareRunSettings` the branch is pre-empted by card 5's `refuseMovedKeys`, which is intended.
- **Card 2, `load` check on default and worktree layers:** reachable. A raw file holding an old key passes `stackLayers` and `EffectiveTemplate`, then reaches the check before `warnUnknownKeys` (`internal/configengine/config.go:218-251`).
- **Card 2, `Set` refusal:** reachable. configcli calls `configengine.Set` directly and checks nothing but the module first (`internal/configcli/configcli.go:210`).
- **Card 2, `Unset` admits an old name:** reachable. It already removes any key (`internal/configengine/set.go:114-136`).
- **Card 4, `CheckSettingKey` refuses a path through a scalar:** reachable.
- **Card 4, `KeyAtLine` deepest-key row:** reachable.
- **Card 5, `refuseMovedKeys` in `PrepareRunSettings`:** reachable from `lym board upsert` (`internal/boardcli/runsettings.go:144`), Seed-Child (`internal/battencli/runsettings.go:43,49`) and `checkProfiles` (`internal/configreg/runsettings.go:164`).
- **Card 5, scalar `discussion` in run_settings:** reachable.
- **Card 5, `TestSeedChild_RunSettings` row:** reaches only Seed-Child's wrapper, through a fake. See finding 4.
- **Card 5, `CheckSettingKey` moved-key refusal at recipe build:** reachable. `tapestry.plan_split` matches the entry-gate operand pattern (`internal/shedbuild/entrygate.go:27`).
- **Card 6, move before prune** (local, worktree, hub base, prime seed): reachable on each path in `internal/configsync/configsync.go:235,227,497,489`. The prime seed is reached only when `primeConfig` parses, since it returns nil otherwise (`configsync.go:386-399`).
- **Card 6, `moveOldKeys` parse-failure `*FileError`:** not reached by the named test. See finding 3.
- **Card 6, `dropped-null` and `dropped-new-set` rows:** reachable, since reconcile reads raw bytes.
- **Card 7, profiles move:** reachable for a present file and its template base. The absent-file case cannot come from an unmoved prime copy. The profiles module is `HubWide`, with no `SeedsFromPrime` and no `OpenMaps` (`internal/configreg/configreg.go:167`), so an absent file seeds from the template (`configsync.go:451`). The guard is complete there.
- **Card 8, the board staying "as it was" after a failed write:** cannot occur. See finding 2.
- **Card 8, an unparseable board giving `skipped`:** reachable, but by a different cause than the one named. See finding 5.
- **Card 8, a skipped hub-wide write runs no carry:** reachable (`internal/hubreconcile/walk.go:268-269`).
- **Card 9, walk before load in the `lym loom` pre-run, including the circling group and a non-repo cwd:** reachable.
- **Card 9, the same guard on other paths:** the guarded load recurs unguarded. See finding 1.

**Findings**

1. **BLOCKING, design. Card 9 (`09-reconcile-before-load.md`).** The walk-before-load guard covers the `lym loom` pre-run plus `hubLandDeps`, `wireHub` and `loomConfigOf`. The same strict loom and tapestry loads recur on verbs it does not guard:
   - `lym shed run|step|status|pause|goto` for every loom-contract recipe arms through `loomcli.ArmAt` (`internal/shedcli/table.go:51-94`). That calls `armAt`, then `wire`, which runs `loomengine.LoadConfig` (`internal/loomcli/wiring.go:598`). It never passes through the loom pre-run (`internal/shedcli/cli.go:165-195`, `internal/loomcli/arm.go:507-510`).
   - Batten verbs other than `run` reach `configreg.SettingLookup`, which runs `configengine.Load` on the tapestry and loom specs (`internal/configreg/settinglookup.go:55`). The call sites are `fixerTimeout` (`internal/battencli/wire.go:807`) and `env.Setting` (`internal/battencli/wire.go:1691`). Only `verb == "run"` reconciles (`internal/battencli/arm.go:301`).
   - `lym fabric` merge verbs load tapestry through `websterInFlightWarnings` (`internal/fabriccli/merge_verbs.go:37`).

   D5 requires every verb outside `lym config` that loads either config to walk first. The card's enumeration follows direct `loomengine.LoadConfig` callers and misses the `ArmAt` and `SettingLookup` routes.

2. **BLOCKING, design. Card 8 (`08-board-run-settings-carry.md`), `RewriteRunSettings` and the `EnsureFailingRunSettingsCarry` row.** The card says `beforeCommit` runs after the save and "the board's write path restores the store it saved, so a failed write leaves the board as it was". The named path, `boardCriticalSection`, does not do that:
   - Its `afterSave` hook runs after `store.Save()`, and an error from it returns with no restore (`internal/boardengine/board.go:181-189`).
   - It never commits or pushes synchronously. It renders, then spawns a detached sync (`board.go:191-199`). So a commit or push failure never reaches `carryMovedRunSettings`.

   The test row's asserted state (entry unchanged after the failed write) cannot occur on this path. D7's acceptance criterion "a walk whose board write fails leaves the entry as it was" is not reachable as planned.

3. **LOW, consistency. Card 6 (`06-reconcile-moves.md`), `TestReconcile_FailureNamesItsFile`.** The card says the unparseable-file rows now get their error "from the move's `*FileError`". Those rows use `batcher`, `shuttle` and a hub file (`internal/configsync/configsync_test.go` around lines 1292-1310). These modules have no move table, and the card says `moveOldKeys` returns such a document unchanged. So the error still comes from the later prune or reconcile. The move's own parse-failure branch, which needs a loom or tapestry file that does not parse, has no reaching row.

4. **LOW, design. Card 5 (`05-moved-key-tables.md`), `TestSeedChild_RunSettings`.** The row's fake `PrepareRunSettings` returns hand-written moved-key text. It checks the wrapper's single `way forward:` against text the test authored, not the real `refuseMovedKeys` output. The real refusal is reached separately by `TestPrepareRunSettings`, which asserts it holds no `way forward:`. So the two-part composition D4 names for Seed-Child is pinned only indirectly.

5. **LOW, design. Card 8, `TestCarryMovedRunSettings_UnreadableBoardIsSkipped`.** The `carryHub` fixture writes only `board.json`, with no board config (`internal/hubreconcile/carry_test.go:19-34`). `RewriteRunSettings` opens the board "from its own board config", and `boardengine.LoadConfig` refuses an absent config file first (`internal/boardengine/config.go:161-168`). The row reaches `skipped` through the missing config, never through "a board that does not parse".

Tool calls:
1. A grep of configcli's set path, the profiles module's registry flags, `runprofile.Load`, the modules in `TestReconcile_FailureNamesItsFile`, `ValidateLocal`'s call site in the layer stack, and shedbuild's setting checks. My context did not show whether any upstream check pre-empts the card 2, 5, 6 and 7 guards.
2. A grep of shedbuild's entry-gate operand pattern and boardengine's board-open and `LoadConfig`. My context lacked how a recipe operand is checked at build and how `RewriteRunSettings`' board open fails in the `carryHub` fixture.
