# Card 3 — `review` and `fix` model lists, picked per round

**Create:**
- `plan:internal/burlerengine#ModelChoice` -> `type ModelChoice struct`
- `plan:internal/burlerengine#RoundModels` -> `type RoundModels struct`
- `plan:internal/burlerengine#RoundModels.Pick` -> `func (m RoundModels) Pick(round int) (review, fix ModelChoice)`
- `plan:internal/shedadapters#BurlerDeps` -> `type BurlerDeps struct`
- `internal/burlerengine/models.go`
- `internal/burlerengine/models_test.go`

**Edit:**
- `internal/loomengine/config.go`
- `internal/loomengine/template.yaml`
- `internal/loomengine/review.go`
- `internal/loomengine/config_test.go`
- `internal/loomengine/review_test.go`
- `internal/configreg/configreg.go`
- `internal/configreg/configreg_test.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_burler.go`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/shedadapters/burler.go`
- `internal/shedadapters/burler_test.go`
- `internal/shedadapters/bouncer_fixture_test.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_test.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `contracts/recipes/loom-recipe.yaml`

**Uses:**
- `internal/batcher#ConfigOpenMaps`
- `internal/loomengine#LoadConfig`

**Intent:**
Implement D9 and D10 while the round is still one agent, so the model plumbing lands before the split: `loom.yaml` gains a `fix` key beside `review`, both accept one model-spec or a per-round list, and the burler round producer picks its model by round.
Until card 6 splits the round, the single agent keeps running on the `review` pick, which is today's model source, so behaviour is unchanged for an existing scalar config.

`burlerengine/models.go`: `ModelChoice` carries `Model`, `Effort` and `Version` (one resolved model-spec);
`RoundModels` carries `Review` and `Fix`, each a `[]ModelChoice`;
`Pick(round)` returns entry `round` of each list (1-based), the last entry for a round past the list, a round below 1 treated as round 1, and a zero `ModelChoice` for an empty list, each list resolved independently.
`models_test.go` is one table-driven test of `Pick` covering those rows.

`loomengine`: add a scalar-or-list YAML type for a model-spec list (its `UnmarshalYAML` accepts a scalar or a block sequence of scalars, rejecting a mapping);
`Config.Review` and the new `Config.Fix` use it.
`LoadConfig` rejects an empty list and parses every entry with `modelspec.Parse`, naming the key and entry index in the error (`PATTERN-config-strictness`: the strict path stays strict).
Add `ConfigOpenMaps()` returning `review` and `fix`, following `batcher.ConfigOpenMaps`, pass it to `configengine.Load` in `LoadConfig`, and register it as the `loom` module's `OpenMaps` in `configreg`, so reconcile, `--set` and the missing-key check carry an operator's list whole instead of treating the scalar template leaf as missing.
An existing scalar `review` loads unchanged as an all-rounds list of one;
a config without `fix` takes the template's value in memory, as every missing key does.
Template: `review: sonnet[medium]` and new `fix: opus[medium]`, each comment saying it is the reviewer's/fixer's model-spec and that a YAML list sets one model per round with the last entry for every later round.
`ResolveReview` resolves every entry of both lists through the registry at run start, so a bad later-round entry fails before round 1, and returns `ReviewSettings{Models burlerengine.RoundModels, Timeout}` in place of the single `Model`/`Effort`/`Version`.

`shedrecipe`: `Env.ReviewModel`/`ReviewEffort`/`ReviewVersion` become `Env.ReviewModels burlerengine.RoundModels`, its doc saying BurlerRound picks from it per round;
`ReviewTimeout` stays.
`burlerRoundEntry` drops the row's `model` and `effort` keys (D10), so `configRejectUnknown` now rejects them, keeps `timeout_s`, and hands `env.ReviewModels` to the producer.
`contracts/recipes/loom-recipe.yaml`: the BurlerRound rows' comments say model choice lives in `loom.yaml`'s `review`/`fix` keys only and the row carries no `model`/`effort` key; Bouncer rows' comments are untouched.

`shedadapters`: `NewBurlerProducer(name, deps BurlerDeps, profile, opts, runDir, now)`, where `BurlerDeps` holds `Runner BurlerRunner`, `Attach Shuttle` (the existing live-round probe seam, unchanged here) and `Models burlerengine.RoundModels`;
the nil-runner and nil-attach errors keep their wording.
`Call` sets each attempt's `RunOpts.Model`/`Effort` from `Models.Pick(round)`'s review choice, using the round it already computes.

`loomcli/wiring.go` fills `Env.ReviewModels` from `ResolveReview`.

Tests: `config_test.go` gains rows for a list `review`, a list `fix`, an empty list, an unparseable entry and a mapping value (load errors), and a scalar `review` loading as a list of one;
`review_test.go` asserts both lists resolve and a bad later entry fails;
`configreg_test.go` asserts the loom module declares the two open maps;
`entries_burler_test.go` asserts a row with `model` or `effort` is rejected and the models reach the producer;
`entries_bouncer_test.go` drops its `env.ReviewModel` assignment;
`burler_test.go` asserts round 1 runs entry 1 and a round past the list runs the last entry;
`bouncer_fixture_test.go`, `smoke_attachprobe_test.go` and `wiring_test.go` move to the new constructor and Env field.
Grep the repository for the old `model`/`effort` BurlerRound keys and every `ReviewModel` assertion, and update each.

**ImpactSummary:** loom config shape, `ResolveReview`'s result, `shedrecipe.Env` and `NewBurlerProducer`'s signature change together; the single round agent keeps the `review` model.
