```yaml
lens: guard-reach
outcome: reported
```

guard-reach fork: checked whether each guard, refusal or test row the cards add can be reached.

Per guard, reachable or not:
- Card 1, `TestLoadConfig_Refuses` subtest with local `code_reviewer_fan: nosuchfan`: reachable. With the key now `CategoryNone`, the local layer admits it. `LoadConfig` then calls `lenses.ValidateFanKeys` (`internal/loomengine/config.go:1225`). The board holds no lenses.yaml, so `nosuchfan` misses the template and the error is `loom config key "code_reviewer_fan" entry 1: …; define the fan in lenses.yaml, or set the entry to ""` (`internal/lenses/fankeys.go`, the `ResolveFan` branch).
- Card 1, `TestLoadConfig_Refuses` local-fan loop with the other five keys: reachable. Their guard category still refuses them in the local layer (`internal/loomengine/config_test.go:685-700`).
- Card 1, `TestLoadConfig_Loads` "hub stack" row for a local `code_reviewer_fan: ""`: reachable. An empty entry skips the lenses read in `ValidateFanKeys`.
- Card 1, `TestModules_LocalLayerRefusals` admitted row: reachable, through `configengine.Layers` (`internal/configreg/configreg_test.go:181-215`).
- Card 1, `TestPrepareRunSettings` darn `darn_reviewer_fan` and `review_fan` rows becoming admitted: reachable. Both key moves exist (`internal/configreg/keymove.go:30,32`), and the guard refusal no longer fires.
- Card 1, `TestPrepareRunSettings` `webster_fan` and `webster_reviewer_fan` rows: reachable. `webster_fan` renames to `webster_reviewer_fan`, then to `code_reviewer_fan` (`internal/loomengine/config.go:1021,1035`). The same two-step shape is what the `webster_review` row shows with its carried comment (`internal/configreg/runsettings_test.go:105`).
- Card 1, `TestPrepareRunSettings` refused rows for `plan_fan`, `discussion_fix_fan` and `plan_fix_fan`: reachable. They rename to `plan_reviewer_fan`, `discussion_fixer_fan` and `plan_fixer_fan` (`config.go:1020,1023,1024`), which stay guards, by the same path as the existing `discussion_fan` row (`runsettings_test.go:97-101`). No upstream check rejects them earlier.
- Card 1, `TestPrepareRunSettings` direct `code_reviewer_fan: ""` row: reachable.
- Card 2, `TestReconcileAll_SparseLayers` local `code_reviewer_fan: ""` row: reachable as a negative pin. `ReconcileAll` calls `reconcileSparseFile` for `LayerLocal` with `strip == nil` (`internal/configsync/configsync.go:424`). Only a non-nil strip reaches `stripToSparse` and `stripFormerValues` (`configsync.go:579-586`), so the former value `code_reviewer_fan: ""` (`configsync.go:69-71`) is never stripped from a local file.
- Card 2, battencli "no profiles file…" row: reachable. The light darn block reaches `PrepareRunSettings`, and after card 1 the open key passes `checkLocalDocument`.
- Card 2, `TestProfilesCheckLoad` template row: reachable. It fails without card 1, and card 1 runs first.
- Card 3, `emphasisLineGaps` rows: all reachable.
  - `leanness` and `scenarios` are not exempt, so they are returned, sorted.
  - `generic` is skipped.
  - `board-scope` is exempt.
  - The assertion that `procedureLenses` holds `board-scope` and lacks `leanness` is reachable.
- Card 3, `TestDiscussionTable` `[board-scope]` rows and the skip in the dropped-lens check: reachable. The template fan now holds `board-scope`, so without the skip the check would flag it.
- Card 3, `burlerOldTemplate` `[:5]`: reachable. It differs from the six-lens template fan.
- Card 4, `writerFanGaps` rows: reachable.
  - A reviewer `way-forward` missing from the writer fan gives one message.
  - `affected-set` absorbed through `forced-edits` gives none.
  - `repo-invariants` with an empty `reviewerOnly` gives one message.
- Card 5's claim that `CheckLoad` under `lens_set: csharp` fails on a leftover `code-pattern-fit`: reachable. The csharp layer names it only in a fan (`internal/lenses/sets/csharp.yaml:53`) and defines no such lens.
- Card 1's bound, "a wrong name fails the pair's loom config load through `ValidateFanKeys`": reachable on loom pairs. On darn and pawl pairs that guarded site recurs unguarded (finding 1).

Findings:

1. LOW, consistency. Card 1 (`01-code-reviewer-fan-local.md`), the Intent's bound paragraph. The card says a wrong `code_reviewer_fan` name from the newly opened local, profile or `run_settings` route "fails the pair's loom config load through `lenses.ValidateFanKeys`". That holds only where `loomengine.LoadConfig` runs: the loom and webster wiring, and configreg's `CheckLoad`.
   - A darn or pawl pair reads `loom.code_reviewer_fan` through `configreg.SettingLookup`, which loads with `configengine.Load` and no fan resolution (`internal/configreg/settinglookup.go:51`).
   - `shedrecipe`'s `fanSetting` checks only the value's shape (`internal/shedrecipe/entries_settings.go:76-106`).
   - So on darn and pawl pairs, D6's target recipes, a bad name passes Seed-Child and the load. It is first refused at the BurlerReview round, when the recorded fan fails to resolve (`internal/burlerengine/doc.go:90`).
   - The card's stated bound names a guard that is not on that path, and no row pins the darn/pawl case.

   Fix: say in the Intent that on a darn or pawl pair the wrong name is refused when the Code-Review round resolves its fan, not at a loom config load. Alternatively, name the test that pins that refusal.

Tool calls:
1. `internal/configsync/configsync.go` `reconcileSparseFile`: whether a local-layer reconcile reaches the former-value strip. My context had the strip but not its call condition.
2. `shedrecipe`'s fan-setting readers and `internal/configreg/settinglookup.go` `SettingLookup`: how a darn or pawl run reads `code_reviewer_fan`. The manifest covered only loom's load path.
3. `fanSetting` in `internal/shedrecipe/entries_settings.go` and configreg's loom `CheckLoad` wiring: whether anything on the setting path resolves the fan name.
4. (No further calls.)
