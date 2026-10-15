# Card 3 — Burler and Bouncer models are shuttle ModelChoices

**Delete:**
- `internal/burlerengine#ModelChoice`

**Edit:**
- `internal/burlerengine#RoundModels`
- `internal/burlerengine#RoundModels.Pick` -> `func (m RoundModels) Pick(round int) (review, fix shuttleengine.ModelChoice)`
- `internal/burlerengine#pickRound` -> `func pickRound(choices []shuttleengine.ModelChoice, round int) shuttleengine.ModelChoice`
- `internal/burlerengine#RunOpts`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#Profile.fixTable`
- `internal/burlerengine/doc.go`
- `internal/burlerengine#TestRoundModels_Pick`
- `internal/burlerengine#TestEngine_Review_TableShape`
- `internal/burlerengine#TestEngine_Fix_TableShapeAndFiles`
- `internal/burlercli#burlerCLI.runCmd`
- `internal/burlercli#TestRunCmd`
- `internal/loomengine#BouncerModels`
- `internal/loomengine#resolveKeyedModelChoice` -> `func resolveKeyedModelChoice(key, raw string, reg modelspec.Registry) (shuttleengine.ModelChoice, error)`
- `internal/loomengine#resolveModelChoices` -> `func resolveModelChoices(key string, specs ModelSpecList, reg modelspec.Registry) ([]shuttleengine.ModelChoice, error)`
- `internal/loomengine#fannedReview` -> `func fannedReview(fan string, fanReview []shuttleengine.ModelChoice) []shuttleengine.ModelChoice`
- `internal/loomengine#resolveSegmentModels` -> `func resolveSegmentModels(segment string, reviewSpecs, fixSpecs ModelSpecList, fanReview []shuttleengine.ModelChoice, runWide burlerengine.RoundModels, reg modelspec.Registry) (burlerengine.RoundModels, error)`
- `internal/loomengine#resolveSegmentBouncerModels`
- `internal/loomengine#ResolveReview`
- `internal/loomengine#TestResolveReview`
- `internal/loomengine#TestResolveJudge`
- `internal/shedadapters#BouncerModels`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#Bouncer.recordPass`
- `internal/shedadapters#newHalfUsage` -> `func newHalfUsage(model shuttleengine.ModelChoice, half burlerengine.Half) halfUsage`
- `internal/shedadapters#newPassUsage` -> `func newPassUsage(model shuttleengine.ModelChoice, result shuttleengine.Result) halfUsage`
- `internal/shedadapters#reviewHalfUsage` -> `func reviewHalfUsage(model shuttleengine.ModelChoice, half burlerengine.Half) halfUsage`
- `internal/shedadapters#BurlerReviewProducer.Call`
- `internal/shedadapters#BurlerReviewProducer.acceptedExit`
- `internal/shedadapters#BurlerReviewProducer.probeLiveReview`
- `internal/shedadapters#BurlerFixProducer.Call`
- `internal/shedadapters#BurlerFixProducer.fixedExit`
- `internal/shedadapters#BurlerFixProducer.probeLiveFix`
- `internal/shedadapters#newBouncerFixture`
- `internal/shedadapters#requirePassEntry`
- `internal/shedadapters#TestBouncer_JudgeCall_Verdicts`
- `internal/shedadapters#TestBouncer_SeedCall_HappyPath`
- `internal/shedadapters#TestBurlerReviewProducer_Call_ProfileAndOpts`
- `internal/shedadapters#TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded`
- `internal/shedadapters#TestBurlerFixProducer_Call_FinishedFixIsRecorded`
- `internal/shedrecipe#TestBouncerEntry_EnvJudgeFallback`
- `internal/shedrecipe#TestBurlerHalfEntries_EnvFallback`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`

**Intent:**
D3 for burler and the Bouncer.
`burlerengine.ModelChoice` is deleted; `RoundModels.Review` and `.Fix` become `[]shuttleengine.ModelChoice`, `RunOpts.Review` and `.Fix` become `shuttleengine.ModelChoice`, and `Pick` and `pickRound` return it, each entry keeping its own engine, so round 2 can run on a different engine from round 1.
`reviewTable` and `fixTable` set their seat's `ModelChoice` to `opts.Review` and `opts.Fix` whole.
`loomengine.BouncerModels` and `shedadapters.BouncerModels` hold `shuttleengine.ModelChoice` seed and judge pairs (the wiring's conversion between the two keeps compiling, their fields staying identical), and the Bouncer's seed and judge specs set `ModelChoice: b.cfg.Models.Seed` and `ModelChoice: b.cfg.Models.Judge`.
`resolveKeyedModelChoice` and `resolveModelChoices` build each entry with `shuttleengine.NewModelChoice(resolved)`, so loom's review, fix, fan-review, judge and seed choices carry their aliases' engines.
The usage builders take the new type; what they write is card 6's.
`lyx burler run`'s `--review-model`, `--review-effort`, `--fix-model` and `--fix-effort` keep taking a provider model string and build `shuttleengine.ModelChoice{Model: ..., Effort: ...}` with `Engine` empty, which is the default engine.
`burlerengine/doc.go` states that the per-round lists carry each entry's engine, and `models.go`'s file header, which says the file declares the per-round model choices, is reworded to say it declares the per-round lists of shuttle model choices.

Tests:
- `TestRoundModels_Pick` gains a row whose review list's entries name engines `a` and `b`: round 1 picks `a`, round 2 and a round past the list pick `b`.
- `TestEngine_Review_TableShape` and `TestEngine_Fix_TableShapeAndFiles` pass a choice with `Engine: "b"` and assert the half's seat carries it.
- `TestResolveReview` gains a row over a registry literal whose aliases name engines `claude` and `other`: a review list of the two yields entries with those engines in order; `TestResolveJudge` asserts the judge's and seed's `Engine`, the seed fallen back to the judge key included.
- `TestBouncer_JudgeCall_Verdicts` and `TestBouncer_SeedCall_HappyPath` give the fixture's judge and seed choices engines `j` and `s` and assert the spawned spec's `Engine` (the fake shuttle's `GotSpec`) is the pass's own.
- `TestRunCmd`'s flag-mapping row also asserts each started half's spec has an empty `Engine`.
- The other listed tests move their `burlerengine.ModelChoice` literals to `shuttleengine.ModelChoice`.

**ImpactSummary:** deletes one burler type; its uses across burlerengine, burlercli, loomengine, shedadapters, shedrecipe and loomcli change type only, and the loom review resolvers now carry each alias's engine.

**Commit:** 3: burler and Bouncer models are shuttle ModelChoices
