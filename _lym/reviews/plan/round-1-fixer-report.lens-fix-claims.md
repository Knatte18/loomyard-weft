```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked every sentence the fix added or rewrote that states a bound, behaviour or fact about the code against HEAD.

**Findings**

- **MEDIUM**, card 1 (`_lym/plan/01-code-reviewer-fan-local.md`), Intent, the added sentence "…so there the wrong name is refused when the Code-Review round resolves the recorded fan, with the same way forward."
  - Summary: on a darn or pawl pair the refusal does not give the same way forward as `ValidateFanKeys`.
  - Evidence: `internal/lenses/fankeys.go:77` (`ValidateFanKeys`) offers "define the fan in lenses.yaml, or set the entry to \"\"".
    The round-time refusal comes from `internal/burlerengine/profile.go:200-204` (`resolveRecordedFan`), which offers "define the fan in lenses.yaml, then resume the run or re-run the verb".
    It never mentions setting the entry to `""`.
  - The reviewer's own wording meant the operator's recovery, which is the same: setting the pair's local entry to `""` or to a defined fan clears it.
    The card's wording instead claims the refusal says this.
  - Suggested rewording: "…refused when the Code-Review round resolves the recorded fan, whose refusal names only defining the fan and resuming, while setting the pair's local entry to `""` or to a fan `lenses.yaml` defines clears it as well."

**Claims that hold**

- Card 1: a webster pair's wiring calls `loomengine.LoadConfig` (`internal/webstercli/wiring.go:125`), as a loom pair's does (`internal/loomcli/wiring.go:378`).
- Card 1: darn and pawl pairs read `loom.code_reviewer_fan` as a recipe `reviewer_fan_setting` (`contracts/recipes/darn-recipe.yaml:127,153`, `pawl-recipe.yaml:70,92`).
  That read goes through `env.Setting`, which is `configreg.SettingLookup` (`internal/battencli/wire.go:1444`, `internal/loomcli/wiring.go:761`), and `shedrecipe`'s `fanSetting` resolves no fan.
  The round resolves the fan in `burlerengine.resolveRecordedFan`.
- Card 4: `internal/lenses/config_test.go:4` holds the quoted header text.
- Cards 3 and 5: `--set` is repeatable (`internal/configcli/doc.go:49`).
  `setChecked` checks the whole set of pairs once, and rolls a write back when the error text changes (`configcli.go:273-296`).
  `lenses.CheckLoad` joins every broken fan into one error (`internal/lenses/config.go:212`), so a partial fix changes the text and is rolled back.

**Tool calls**

1. Searched for `SettingLookup` and `code_reviewer_fan` readers in the darn and pawl packages, to learn which path reads the key; my context did not show it.
2. Searched for the non-test sites of `code_reviewer_fan` and read `loomcli/wiring.go:750-770` and `battencli/wire.go:1435-1450`, for the wiring of `env.Setting`.
3. Searched the recipe YAML for `code_reviewer_fan`, to find how darn and pawl reach the key.
4. Searched for the readers of `reviewer_fan_setting`, `ResolveFan` and `errors.Join` in `lenses`, to find where a fan is resolved and whether `CheckLoad` joins.
5. Searched `fanSetting` and the callers of `ResolveFan`, to find where the round resolves the fan.
6. Read `burlerengine/profile.go:195-215` and `lenses/fankeys.go:70-90`, to compare the two refusals' ways forward.
