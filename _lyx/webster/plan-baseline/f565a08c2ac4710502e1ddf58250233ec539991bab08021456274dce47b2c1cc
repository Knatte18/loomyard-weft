# Card 3 — bouncer-seed-judge-split

**Create:**
- `internal/shedadapters#BouncerModels`

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedadapters#newBouncerFixture`
- `internal/shedadapters#TestBouncer_SeedCall_HappyPath`
- `internal/shedadapters#TestBouncer_JudgeCall_Verdicts`
- `internal/shedadapters#TestNewBouncer_AcceptedConfigs`

**Prosa:**
- `internal/shedadapters/doc.go`

**Intent:** Give the Bouncer separate seed and judge model settings (D4's config side), without changing where the values come from yet.
`BouncerModels` holds two `burlerengine.ModelChoice` fields, `Seed` and `Judge`, each an already-resolved model, effort and version triple.
`BouncerConfig` replaces its `Model`, `Effort` and `Version` fields with one field `Models BouncerModels`; both pairs stay accepted empty, deferring to the provider default, so `NewBouncer` validates nothing new.
`runSeedSpawn` fills its spec from `Models.Seed` and `judgeCall` from `Models.Judge`.
`bouncerEntry` keeps reading the row's `model`, `effort` and `version` keys and the Env fallback as today, and fills both `Seed` and `Judge` from that one triple, so this card changes no spawned model; card 4 retires the keys and threads the per-segment values.
`newBouncerFixture` sets distinct seed and judge triples; `TestBouncer_SeedCall_HappyPath` asserts the seed spec carries the seed triple, and `TestBouncer_JudgeCall_Verdicts` gains assertions that the judge spec carries the judge triple and not the seed's.
`TestNewBouncer_AcceptedConfigs` keeps its empty-triple row over the new field.
In `doc.go`, the sentence naming the Bouncer's told inputs says the resolved seed and judge pairs instead of one triple.

**ImpactSummary:** A field change on BouncerConfig with one production writer, shedrecipe's bouncerEntry, and the Bouncer's two spawn sites as readers; no model any pass runs on changes.
