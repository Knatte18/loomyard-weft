# Card 6 — round-N-usage.yaml records each half's and pass's engine

**Edit:**
- `internal/burlerengine#Half`
- `internal/burlerengine#halfOf`
- `internal/burlerengine#TestEngine_Review_Outcomes`
- `internal/shedadapters#halfUsage`
- `internal/shedadapters#newHalfUsage`
- `internal/shedadapters#newPassUsage`
- `internal/shedadapters/doc.go`
- `internal/shedadapters#requirePassEntry`
- `internal/shedadapters#TestBouncer_UsageRecord`
- `internal/shedadapters#TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded`
- `internal/shedadapters#TestBurlerFixProducer_Call_FinishedFixIsRecorded`

**Intent:**
D4's usage record.
`burlerengine.Half` gains `Engine`, which `halfOf` copies from the half's `shuttleengine.Result.Engine`; a half that never started (`Half{StartError: ...}`) has no run record and keeps it empty.
`halfUsage` gains `Engine string` under the yaml key `engine`, with `omitempty`, beside `model` and `effort`.
`newHalfUsage` writes it from `half.Engine`, never from the choice, so it names the engine that ran even when the choice left `Engine` empty; `newPassUsage` copies `result.Engine` into the half it builds, so a Bouncer pass records the engine its run record names.
A half or pass with no run record writes no `engine` key; a record written before this change decodes with the field empty (`PATTERN-tolerant-records`: the struct decode ignores no key it lacks and refuses none it gains).
`shedadapters/doc.go`'s usage-record paragraph names the engine beside each half's model.

Tests:
- `TestBouncer_UsageRecord`'s known result carries `Engine: "claude"` while the fixture's seed and judge choices leave `Engine` empty; `requirePassEntry` asserts the entry's `Engine` is `claude`, so the seed and judge rows prove a choice with an empty engine records the run's engine; the `a Run error records nothing` row stays.
- `TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded` and `TestBurlerFixProducer_Call_FinishedFixIsRecorded` script the half's `Engine` and assert it in the read record; the review test gains a row whose half has no engine and asserts the written file holds no `engine:` key.
- `TestEngine_Review_Outcomes`' done row asserts `ReviewResult.Review.Engine` equals the scripted result's engine.

**ImpactSummary:** one added field on `Half` and on the usage entry, filled by the three usage builders; the record stays additive and readers ignore the new key.

**Commit:** 6: round-N-usage.yaml records each half's and pass's engine
