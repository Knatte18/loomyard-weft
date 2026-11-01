```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
1. **Template `active:` names `wide`.** Pinned by `TestActive` in `internal/batcher/config_test.go`: the `HasPrefix` check on `active: "wide"`, the `templateDefaultResolvesWide` row, and the two absent-base rows that want `wide`. It would fail if the template named another profile.
2. **`wide` is the cost profile with `budget: 300000`.** Only partly pinned. `nonPositiveBudgetErrors` replaces the literal `budget: 300000`, so it fails if `wide`'s budget changes. It does not check the value itself: nothing asserts that `wide`'s budget is 300000 rather than some other positive number, because it only edits the string already there.
3. **`wide` carries the same weights as `cautious`.** Pinned by `TestProfileWeights/templateProfilesShareTheMeasuredWeights`, which compares the two `Weights` structs and checks `wide`'s orientation and batch_growth. It does not check `max_cards`, `batchifier` or `cautious`'s budget.
4. **`cautious` stays at budget 200000 and `identity` is unchanged.** No test row. No test asserts `cautious`'s budget or `max_cards`, so a change to `cautious`'s budget or `max_cards` would pass. The identity profile is pinned only indirectly, through the `emptyActive…` rows.
5. **The former-default strip of `active: "cautious"` on reconcile.** No test row for the added `formerTemplateValues["batcher"]["active"]` entry.
   - `TestFormerTemplateValues_DifferFromTheTemplate` (`configsync_test.go:1699`) only checks that the former value exists as a template unit and differs from the current value. It does not check that a hub file holding `active: "cautious"` is stripped.
   - The batcher row in `TestReconcileAll_MigratesToSparse` (`configsync_test.go:1543`) uses `batcherDiffering`, which is the current template with orientation changed. There `active` is stripped as equal to the template (`wide`), not as the former `cautious` value.
   - Deleting the new `"active"` entry from the map would therefore leave every test green.
   - The "marked board keeps a former value as a pin" behaviour for `active` is likewise untested. `configsync_test.go:1043` and `:1527` only cover other modules.
6. **A deliberately chosen `cautious` after the former-default strip.** No test, and no stripping test for an `active:` that is absent or not a string. The diff does not touch that path, but its only coverage today is the generic strip.
7. **`TestReconcileAll_MigratesToSparse` `wantStripped` rewrite.** It gained the `profiles.wide.*` and `profiles.cautious.weights.orientation` entries and lost none of the earlier ones. No coverage was lost; the `wide` entries are the new coverage.
8. **`masterBaseRewrites` rewritten to `profiles.wide…` (`TestReconcileAll`).** Pinned: the retired `master_base` fixture is derived from the template, so the rewrite now lands on the `wide` profile. `MigrateConfig` itself is still pinned on `cautious` by the `config_test.go` rows at lines 432–479.
9. **`MigrateConfig` takes its orientation from the `cautious` profile (`templateOrientation`, `config.go:246`).** Unchanged by the diff, and still pinned by the migration rows at `config_test.go:446–460`.
10. **`repoDefaultTuningOneKeyKeepsTheTemplatesOthers` and `repoDefaultTunedKeyReachesTheProfile`.** Both were rewritten to tune `wide`, with budget 250000 in place of the old 300000. Both rows still cover what they covered before. Without a `cautious`-tuning row, a layer overriding `cautious` is no longer tested through `Active`, though that was never the point of the rows.
11. **`TestPersistentPreRunE_BatcherSelection` swaps `"cautious"` for `"wide"` in its replace.** Pinned. The same test's comment line was updated and the case still runs. The `strings.Replace` would silently no-op if the template stopped saying `wide`, but the `bogus` step would then fail on its own assertion.
12. **`cost_test.go` and `websterengine` fixtures that still use the name `cautious`.** They take profile names as input and never read the template, so they are unaffected.

**Findings**
- **MEDIUM, `design`, `internal/configsync/configsync_test.go` (near `:1543`, `TestReconcileAll_MigratesToSparse`):** the new `formerTemplateValues["batcher"]["active"] = "cautious"` strip has no row. Add a case where a hub `batcher.yaml` holds `active: "cautious"` and nothing else, expect `active` reported stripped and the file deleted or empty, and expect resolution to change to `wide`. Add a marked-board row showing the value is kept as a pin. Evidence: `configsync.go:52`; the only batcher row at `configsync_test.go:1543–1558` uses `batcherDiffering`.
- **LOW, `design`, `internal/batcher/config_test.go:403–421`:** the `cautious` profile's budget, `max_cards` and kind, and `wide`'s budget value, are asserted nowhere. The weights comparison covers only `Weights`. Add the budget values and `max_cards` to the profile-shared test, or a row asserting `cautious` resolves at budget 200000 and `wide` at 300000.

**Tool calls**
1. Grep of `configsync_test.go` for former-value tests and a read of lines 1520–1560. The context lacked the neighbouring rows of the migrate test.
2. Grep for `batcherDiffering`. The context lacked its definition, to see whether `active` is stripped as a former value or as equal to the template.
