# Card 6 — burler-half-usage

**Edit:**
- `internal/burlerengine#Half`
- `internal/burlerengine#halfOf`
- `internal/burlerengine#Result`
- `internal/burlerengine#Engine.decideRound`
- `internal/burlerengine#TestEngine_Run_ShuttleOutcomes`
- `internal/burlerengine#TestEngine_Run_ForkSubagentsSpecWiring`
- `internal/burlerengine/doc.go`

**Intent:**
Carries card 4's per-run reading out of a burler round, for decision D8's usage record.
`Half` gains `StartedAt`, `EndedAt` and `Usage shuttleengine.SessionUsage`, and `halfOf` copies them from the half's terminal `shuttleengine.Result`, so a spawned and a resumed round (both reach `decideRound`) carry them alike.
`Result` gains `Lenses []string`: the names of the lenses the round's reviewer ran, in fan order after exclusion, taken from the profile's resolved cluster lenses in `decideRound`, and nil for a solo round.
No outcome, error or audit decision reads the new fields.

No new test: `TestEngine_Run_ShuttleOutcomes` already pins each half's passthrough from the terminal `shuttleengine.Result`, and gains assertions that each half's `StartedAt`, `EndedAt` and `Usage` match the fake shuttle's scripted `Result`;
`TestEngine_Run_ForkSubagentsSpecWiring` already table-drives a done cluster round against a plain one, and gains a `wantLenses` column: the resolved lens names for the cluster row and nil for the plain row.
Which lenses survive a `ClusterExclude` stays pinned by `TestProfileValidate_ClusterExclude`.
The `burlerengine` package doc's `Result` description names the new fields.

**ImpactSummary:** Half and Result gain fields filled in halfOf and decideRound; callers in shedadapters keep compiling and read them from card 7 on.
