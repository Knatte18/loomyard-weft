# Card 7 — round-usage-record

**Create:**
- `plan:internal/shedadapters#roundUsage` -> `type roundUsage struct`
- `plan:internal/shedadapters#halfUsage` -> `type halfUsage struct`
- `plan:internal/shedadapters#roundUsagePath` -> `func roundUsagePath(runDir string, n int) string`
- `plan:internal/shedadapters#newRoundUsage` -> `func newRoundUsage(fan string, review, fix burlerengine.ModelChoice, result burlerengine.Result) roundUsage`
- `plan:internal/shedadapters#writeRoundUsage` -> `func writeRoundUsage(runDir string, round int, u roundUsage) error`
- `plan:internal/shedadapters#readRoundUsage` -> `func readRoundUsage(runDir string, round int) (roundUsage, error)`

**Edit:**
- `internal/shedadapters#BurlerProducer.doneExit`
- `internal/shedadapters/burler_test.go`
- `internal/shedadapters/doc.go`

**Intent:**
Decision D8: every completed review round, solo and fanned, in every review segment, leaves `round-<N>-usage.yaml` beside its review and fixer report in the run directory, which is durable under `_lyx` (`PATTERN-durable-vs-ephemeral-state`).
The members live in a new `internal/shedadapters/roundusage.go`.
`roundUsage` holds the round, the fan name (empty when solo), the lenses the reviewer ran (`burlerengine.Result.Lenses`), and one `halfUsage` per half.
`halfUsage` holds the model and effort, start, end, wall time in seconds, whether the tokens are known, fresh tokens and cache-read tokens;
the reviewer's half also holds the fork count, the forks' fresh and cache-read tokens and the forks' share of the reviewer's fresh tokens.
An unknown reading renders its token fields as unknown rather than zero, and a zero start or end leaves the wall time unknown.
`newRoundUsage` builds the record from the round's fan, the two `ModelChoice` values the round ran on and the round's `burlerengine.Result`;
`writeRoundUsage` renders it as YAML at `roundUsagePath` and `readRoundUsage` decodes it back, so card 8's exclusion eligibility and card 9's facts read the lenses each round ran.
`BurlerProducer.doneExit` writes the record on its success return only, from `p.profile.ClusterFan` and `p.models.Pick(round)`;
a write failure is a warning and never changes or delays the round's outcome.
A gate-failed round writes no record, since it is archived and re-run.

`burler_test.go` gains one table-driven test of the record at the producer surface, read back through `readRoundUsage`: a successful fanned round leaves it with the fan, models and lenses, a solo round with an empty fan and nil lenses, a round whose halves carry an unknown reading and zero times renders tokens and wall time as unknown rather than zero, a gate-failed round leaves none, and an unwritable record leaves the round's `Stuck` hand-off unchanged.
No separate `roundusage.go` test: the write and read helpers are reached only through that table.
The `shedadapters` package doc lists the usage record among the run directory's files.

**ImpactSummary:** doneExit gains a best-effort record write on its success branch; spawned and resumed rounds both pass through it, and no outcome changes.
