# Card 3 — shedrecipe-segment-settings

**Create:**
- `plan:internal/shedrecipe#SegmentSettings` -> `type SegmentSettings struct`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/testkit/envkit#nilLegal`
- `contracts/specs/shed-recipe-spec.md`

**Intent:** Add the segment-settings type and the two new `Env` fields that later cards fill and read; no caller fills them and no entry reads them yet, so behavior is unchanged.

`SegmentSettings` (D1) holds one review segment's resolved values, each field documented:
- `Models burlerengine.RoundModels`, the reviewer and fixer round models, already fallen back to the shared `reviewer_model` and `fixer_model`;
- `ReviewKey` and `FixKey shuttleengine.ModelKey`, the config keys those models came from;
- `Bouncer shedadapters.BouncerModels`, the Bouncer's seed and judge pair;
- `MaxBounces int`, the resolved bounce budget, 0 meaning zero rounds, and `MaxBouncesKey string`, the key it resolved from;
- `HardMaxBounces int`, the resolved hard cap as a round number, 0 meaning none, and `HardMaxBouncesKey string`;
- `ConfigFile string`, the file both keys live in (`loom.yaml` or `darn.yaml`).

`Env` gains, each with a doc comment:
- `Segments map[string]SegmentSettings`, keyed by the recipe's `segment:` value, read by the Bouncer, BurlerReview and BurlerFix entries and by `internal/loomrecipe`; its comment states that it is legal on `Env` because a segment's three rows share one entry;
- `Setting func(module, key string) (any, error)`, the told setting lookup a row reaches by naming `module.key`, returning the module's resolved value from the pair's config stack; nil is legal for a caller whose recipe names no setting.
`Env`'s opening comment ("never a value that differs between two rows") gains that `Segments` is keyed by segment and `Setting` by a row's own reference.
The `ReviewModels` comment's "Env carries roots and run-wide values only" drops `only`, since `Segments` now holds per-segment values; card 8 deletes that comment with its field.
`contracts/specs/shed-recipe-spec.md`'s sentence beginning "The rule that makes the `Env`-versus-`Config` split decidable" is reworded to the new rule: `Env` holds roots, run-wide values and the per-segment settings of `Segments`, keyed by segment so a segment's rows share one entry, and anything per-row is a relative path or scalar in `Config`, resolved against one of those roots by the entry that reads it.

`envkit`'s `nilLegal` gains `Segments` and `Setting` on this card, beside the fields: their nil is legal for a recipe with no review segment or no named setting, so `TestFullEnv_FillsEverySeam` stays green while `FullEnv` fills neither, and `battencli`'s wiring test, which fills neither, needs no change.
No test changes: the existing `envkit` and `battencli` seam tests cover the two legal nils.

**ImpactSummary:** One new exported shedrecipe type and two new Env fields with no filler or reader yet, two nilLegal entries in the test kit, and the shed recipe spec's Env-versus-Config rule reworded to match; no behavior changes.

**Commit:** 3: shedrecipe: add SegmentSettings and the Env.Segments and Env.Setting fields
