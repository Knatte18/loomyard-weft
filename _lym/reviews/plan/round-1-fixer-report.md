# Fixer report — plan, round 1

## Changes

- **F1** (LOW), fixed as proposed, in `_lym/plan/01-code-reviewer-fan-local.md`:
  - Dropped the `TestLoadConfig_Refuses` sibling subtest writing a local `code_reviewer_fan: nosuchfan`; the key-loop change stays.
  - Removed the Edit line `internal/configreg#TestModules_LocalLayerRefusals` and its Tests bullet.
  - Dropped the sentence adding refused `TestPrepareRunSettings` rows for `plan_fan`, `discussion_fix_fan` and `plan_fix_fan`.
- **F3** (LOW), fixed as proposed: `_lym/plan/04-repo-invariants-lens.md`'s Tests in `internal/lenses` gain a bullet changing `config_test.go`'s file header to "each writer fan holding its reviewer fan's lenses but the reviewer-only repo-invariants". No target added.
- **F4** (NIT), fixed as proposed, in the recovery paragraph of `_lym/plan/03-board-scope-lens.md` and of `_lym/plan/05-code-review-lenses.md`: a sentence after the recovery says the `--set` route fixes every fan naming the lens in one call, one `--set` per fan, since a `--set` leaving another broken fan changes the load error and is rolled back.
  Placed as its own sentence rather than inserted mid-sentence, so card 3's "as the record's open risk says" stays attached to the recovery.
  `--set` is repeatable and checked once per call (`internal/configcli/doc.go:49`, `:66`).
- Self-check fixes, in `_lym/plan/01-code-reviewer-fan-local.md`:
  - The `TestPrepareRunSettings` bullet gains a sentence naming what still pins the other five fan keys refused through run settings: the existing `discussion_fan` and `review_fix_fan` refused rows, and the `LocalDenied` set `wantLocalDenied` pins, which `PrepareRunSettings` refuses through.
  - The ImpactSummary reads "run-settings and loom load rows change or join", dropping "local-layer" now that no `TestModules_LocalLayerRefusals` row joins.

## Disputed

- **F2**, `premise false`.
  The finding says a darn or pawl pair meets a wrong `code_reviewer_fan` only when the Code-Review round resolves its fan, since it reads the key through `configreg.SettingLookup`.
  A darn or pawl pair's run is driven by loomcli: `(*loomCLI).wire` serves every recipe, branching on `c.recipe == shedrun.RecipeDarn` (`internal/loomcli/wiring.go:817`) and `c.recipe == shedrun.RecipePawl` (`internal/loomcli/wiring.go:824`), and it calls `loomengine.LoadConfig(fabricengine.ConfigDirs(location), "loom")` unconditionally at `internal/loomcli/wiring.go:378`, whose `lenses.ValidateFanKeys` (`internal/loomengine/config.go:1225`) refuses the wrong name.
  `loomcli/drivewiring.go:119-121` dispatches the darn and pawl recipes there too.
  The `SettingLookup` reads are recipe-row setting reads on top of that load, not a route around it, so card 1's bound, a wrong name fails the pair's loom config load through `ValidateFanKeys`, holds on every pair kind.
  I first added the proposed sentence, then the self-check's fix-proposal fork questioned the split, the code above confirmed it was false, and the sentence was removed; card 1's Intent is as it was before the round on this point.

## Deferred

None.

## Self-check

- fix-consistency, LOW, card 1 no longer names the pins for the other five fan keys staying refused through run settings: fixed as proposed (the sentence added to the `TestPrepareRunSettings` bullet).
- fix-consistency, LOW, card 1's ImpactSummary still says local-layer rows join: fixed as proposed.
- fix-proposal, LOW, the F2 darn/pawl sentence may be wrong if darn and pawl runs go through loomcli's `wire`: fixed with my own change, removing the sentence and disputing F2 rather than narrowing the sentence, since every darn and pawl run goes through `internal/loomcli/wiring.go:378` and no route skips `LoadConfig`.
- fix-claims, MEDIUM, the F2 sentence's "with the same way forward" is false against `burlerengine.resolveRecordedFan`'s refusal: fixed with my own change, the same removal of the sentence, in place of the proposed rewording, which would still assert the false darn/pawl split.
- fix-affected-set: no findings.
