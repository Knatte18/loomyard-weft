# Card 4 — per-segment-bouncer-threading

**Create:**
- `internal/loomengine#BouncerModels`

**Edit:**
- `internal/loomengine#JudgeSettings`
- `internal/loomengine#ResolveJudge`
- `internal/loomengine#TestResolveJudge`
- `internal/loomengine#TestResolveReviewAndJudge_MalformedSpec`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#TestBouncerEntry_EnvJudgeFallback`
- `internal/shedrecipe#TestBouncerEntry_ConstructionFailures`
- `internal/shedrecipe#TestBouncerEntry_RunDirectory`
- `internal/shedrecipe#TestBurlerRoundEntry_RunDirectory`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/testkit/envkit#FullEnv`
- `internal/shedbuild#TestBuild_EveryRegisteredEngineBuilds`
- `contracts/recipes/loom-recipe.yaml`

**Intent:** Implement D1, D2, D4 and D5: resolve each review segment's judge and seed models once at run start and thread them to that segment's Bouncer row, and retire the row's recipe-level model keys.
`loomengine.BouncerModels` holds `Seed` and `Judge`, each a `burlerengine.ModelChoice`, with the same field names and types as `shedadapters.BouncerModels` so the wiring converts one to the other without copying fields.
`JudgeSettings` drops `Model`, `Effort` and `Version` and gains `Discussion`, `Plan` and `Webster`, each a `BouncerModels`.
`ResolveJudge` keeps its signature and resolves, per segment: the judge from the segment's own judge key when non-empty, else `judge`; the seed from the segment's own seed key when non-empty, else `seed` when non-empty, else that segment's resolved judge spec.
Every key is resolved through the registry, and an error names the key the failing value came from, so an unknown alias in any of them fails the run before round 1.
`TestResolveJudge` becomes a table over the acceptance criteria: all keys empty resolves every segment's judge and seed to `judge`; `seed` set moves every seed; `plan_judge` alone moves Plan-Review's judge and seed together and leaves the other two segments on `judge`; `plan_judge` with `seed` or `plan_seed` keeps Plan-Review's seed off the judge; `webster_seed` alone moves only Webster-Review's seed; `seed` and `plan_seed` together put Plan-Review's seed on `plan_seed` and the others on `seed`.
`TestResolveReviewAndJudge_MalformedSpec` gains a row per new judge and seed key, each asserting the error names that key.
`Env` replaces `JudgeModel`, `JudgeEffort` and `JudgeVersion` with `RowBouncerModels map[string]shedadapters.BouncerModels`, keyed by row name as `RowReviewModels` and `RowClusterFans` are, documented as the resolved seed and judge pair of each Bouncer row.
`bouncerEntry` no longer accepts `model`, `effort` or `version`: they leave the `configRejectUnknown` list, so a recipe naming one is refused as an unknown key, as D5 decides.
It fills `BouncerConfig.Models` from `env.RowBouncerModels[name]`, and a row with no entry is a construction error naming the row and the Env field, never a fallback.
`loomCLI.wire` fills the map for `loomshed.NameDiscussionBouncer`, `NamePlanBouncer` and `NameWebsterBouncer` from `JudgeSettings`' three segments, converting each to `shedadapters.BouncerModels`.
`TestWire_ReviewKeysReachTheEnv` seeds `plan_judge`, `seed` and `webster_seed` beside its existing keys and asserts each Bouncer row's map entry: Plan-Bouncer's judge and seed, Webster-Bouncer's seed, and Discussion-Bouncer's judge and seed on `judge` and `seed`.
`envkit.FullEnv` fills the map for the three loom Bouncer rows so every consumer fixture keeps building; `TestBuild_EveryRegisteredEngineBuilds` adds an entry for its own one-row Bouncer name before building.
`shedrecipe.newTestEnv` fills entries for the row names the package's tests pass to `bouncerEntry`, including the conformance test's row name, so every existing construction keeps building.
`TestBouncerEntry_EnvJudgeFallback` is rewritten over the same name: a row takes its map entry's seed triple on the seed spawn and its judge triple on the judge spawn (a second Call over a written round-1 report); `TestBouncerEntry_ConstructionFailures` gains the no-entry row and one row each for a row naming `model`, `effort` or `version`, refused naming the key; `TestBouncerEntry_RunDirectory` and `TestBurlerRoundEntry_RunDirectory` set entries for the rows they construct where `newTestEnv` does not already.
In `contracts/recipes/loom-recipe.yaml`, each Bouncer row's comment saying no `model`/`effort`/`version` key is present is reworded: the keys are not accepted, and `loom.yaml`'s `judge`, `seed` and per-segment keys are the only source of a Bouncer pass's model.

**ImpactSummary:** JudgeSettings' shape and three Env fields change, with loomcli's wire as the one production writer and bouncerEntry as the one reader; with every new key empty each Bouncer pass still spawns on `judge`.
