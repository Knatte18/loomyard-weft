```yaml
lens: row-validity
outcome: reported
```

Row-validity review: checking that each planned test row reaches the branch it claims and fails if the code is wrong, and that no new test repeats an existing one.

**Findings**

1. **BLOCKING — Card 9, `TestArm_StaleStampReconcilesBeforeANonStartVerbLoads`.** The row cannot reach the code card 9 changes.
   - The card's example verb is `lym loom circling`, but `resolvePersistentPreRun` skips `c.arm` for the circling, decision, parent-review and peers groups. Each of those resolves its own worktree in its own `PersistentPreRunE`. Evidence: `internal/loomcli/cli.go:260-283` (`inReviewGroup(cmd)` returns before `c.arm`).
   - `perchHardCap` (`internal/loomcli/circling.go:253`) loads loom and tapestry config with no reconcile before it. Widening `loomCLI.arm` does nothing for these verbs.
   - Under card 2, the strict load would refuse the hub base `code_max_bounces: 2`. So the row's "resolves to 2 and does not halt" fails as written.
   - The same gap is in production: `circling.go:262/267` and the parent-review deps are D5 load sites that no card lists.
   - Fix: add `EnsureOrWarn` to the review groups' pre-runs, or to `perchHardCap` and its siblings, and list them on card 9. Or drive the test through a verb that goes through `arm`.

2. **MEDIUM — Card 5, the D4 "profile block" row.** It is put under `TestPrepareRunSettings`, but it is described as reaching the refusal "through `LayerRunSettings`". That is `internal/configreg#TestLayerRunSettings`, which card 5 does not list (only card 4 edits it). The D4 profile site is really `checkProfiles`, whose test is `internal/configreg#TestProfilesCheckLoad`, and no card lists that test with a moved-key row. The profile-block refusal of D4 therefore has no row on the card that brings it in.

3. **MEDIUM — Card 5, D4's "Seed-Child's refusal prints one way forward".** No planned row reaches Seed-Child's wrapper. The wrapper is in `internal/battenshed/seamchild.go:152-156`, which appends `way forward: correct ... lym board upsert ... lym config profiles`. No `battenshed` or `battencli` test is listed with a moved-key `run_settings` row asserting that exactly one `way forward:` appears. The `TestPrepareRunSettings` row checks only that `PrepareRunSettings` adds no clause of its own.

4. **MEDIUM — Card 5, `TestLoadConfig_Refuses` rows for the local, worktree and hub base layers.** The table has the fields `{name, key, value, wantIn, lensesYAML}` and writes one file (`internal/loomengine/config_test.go:681-687`). It has no field to choose the layer, so the three layer rows, the local `discussion: {reviewer_fan: ""}` Guard row and the local `plan_max_bounces` row cannot be written without new table fields and fixture code. The card does not state those. The same applies to card 4's `TestLoadConfig_Loads` "hub base loom.yaml" row (`config_test.go:165-183`), which loads one fixture.

5. **MEDIUM — Card 2, `TestLoad_Refusals` rows for the default layer and the worktree layer.** The table holds one `content` written to one base, switched to the board dir by `boardDir` (`internal/configengine/config_test.go:257-268`). It cannot stack a worktree file over a default base, so the worktree-layer branch of the new `load` check is not reachable from this table. It also has no `movedKeys` field. The layered fixture (`layerFixture`, `newLayerFixture`) belongs to `internal/configengine_test#TestLoad_LayerStack`, which is the host that can reach both layers. The card does not list it.

6. **LOW — Card 2, `TestSet`.** The table has no `movedKeys` field (`internal/configengine/set_test.go:28-53`). The row needs a new spec field in the table, and the card does not mention it. The same applies to `TestValidateLocal`, which builds its specs inline, where it is a straightforward addition.

7. **LOW — Card 2, the `TestValidateLocal` row `area: {key: 1}`.** The expected result is "passes on to the template's own rules", which states no assertion. Unless the row pins admitted or refused against a template that holds `area.key`, it would pass whether the old-key match wrongly fires on a mapping or not.

8. **LOW — Card 6, `TestReconcileAll_Idempotent` row.** The test is not a table. It reconciles an empty standalone dir twice (`internal/configsync/configsync_test.go:387-412`), so a "moved files" row does not fit. Asserting the second pass inside the `TestReconcileAll_SparseLayers` rows covers the same thing without restructuring.

9. **LOW — Card 6, `TestReconcileHubWideAt_PrunesUnknownKeysOnlyWhenAsked` row.** The table always writes the `landing` module (`configsync_test.go:1235-1236`), and `landing` has no move table. A loom row needs a module field, which the card does not state.

10. **LOW — Card 4, the `TestSettingLookup` row.** It also claims "a `loomrecipe` build of the shipped arras recipe passes". That is not part of `TestSettingLookup`, and the existing `loomrecipe` recipe-build tests already reach it once the arras recipe names `loom.split.reviewer_fan`. Drop the second half as a repeat.

11. **LOW — Card 8, D7 "a failed read".** Only the write failure is reached, through the `beforeRunSettingsCarry` hook. The read-failure branch of `carryMovedRunSettings` has no row. The tier0 `carryHub` pattern (`internal/hubreconcile/carry_test.go:17-33`, with `{not json` boards) would reach it cheaply.

12. **LOW — Card 7, `TestMoveSettings`.** Its "second pass changes nothing" row repeats `TestMoveKeys`' idempotence rows (card 3), since `MoveSettings` is a thin per-module wrapper over `MoveKeys`. Keep only the module-keyed and prefix rows.

**Tool calls**
- `configengine` `TestLoad_Refusals`, `TestSet` and `TestValidateLocal` table heads: needed to see whether card 2's rows fit those tables; my context had only their names.
- `loomcli` arm call sites, circling, the test files that call `arm`, and the `hubforge` and stale-hub fixtures: needed to see whether card 9's verb goes through `arm`.
- `circlingCmd` body and `resolvePersistentPreRun`: needed to confirm that the circling group skips `arm`.
- `hubreconcile/carry_test.go` and the `configsync` `Idempotent` and `PrunesUnknownKeys` heads: needed to check card 6's and card 8's hosts.
- `loomengine` `TestLoadConfig_Refuses` and `TestLoadConfig_Loads` heads, and the module `PrunesUnknownKeys` writes: needed to check whether card 4's and card 5's rows can choose a layer.
