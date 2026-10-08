# Card 6 — burler-half-usage

**Edit:**
- `internal/burlerengine#Half`
- `internal/burlerengine#halfOf`
- `internal/burlerengine#Result`
- `internal/burlerengine#Engine.decideRound`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/doc.go`

**Intent:**
Carries card 4's per-run reading out of a burler round, for decision D8's usage record.
`Half` gains `StartedAt`, `EndedAt` and `Usage shuttleengine.SessionUsage`, and `halfOf` copies them from the half's terminal `shuttleengine.Result`, so a spawned and a resumed round (both reach `decideRound`) carry them alike.
`Result` gains `Lenses []string`: the names of the lenses the round's reviewer ran, in fan order after exclusion, taken from the profile's resolved cluster lenses in `decideRound`, and nil for a solo round.
No outcome, error or audit decision reads the new fields.

`engine_test.go` gains a test that a done cluster round's `Result` carries both halves' usage and times as the fake shuttle reported them and the lenses that survived a `ClusterExclude`, and that a solo round's `Lenses` is nil.
The `burlerengine` package doc's `Result` description names the new fields.

**ImpactSummary:** Half and Result gain fields filled in halfOf and decideRound; callers in shedadapters keep compiling and read them from card 7 on.
