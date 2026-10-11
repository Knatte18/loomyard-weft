# Card 4 — run time breakdown

**Create:**
- `plan:tools/tokencount#RunTime` -> `type RunTime struct`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#RunTally.SetTime` -> `func (run *RunTally) SetTime(rec *ShedRecord)`
- `plan:tools/tokencount#spanOf` -> `func spanOf(run RunTally, rec *ShedRecord) Interval`
- `plan:tools/tokencount#producerTimes` -> `func producerTimes(rec *ShedRecord, span Interval, work []Interval) []ProducerTime`
- `tools/tokencount/runtime_test.go`

**Edit:**
- `tools/tokencount#RunTally`
- `tools/tokencount#CountRun`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#Interval`
- `plan:tools/tokencount#unionOf`
- `plan:tools/tokencount#clipTo`
- `plan:tools/tokencount#totalOf`
- `plan:tools/tokencount#ShedRecord`
- `plan:tools/tokencount#ShedEntry`
- `tools/tokencount#CountOrch`
- `tools/tokencount#writeLines`
- `tools/tokencount#timedAssistant`
- `plan:tools/tokencount#stampedLine`

**Intent:**
Decisions 3 to 6: the run's span, each role's time inside it, the idle time and the Producers rows, computed from Card 2's `Work` and Card 3's `ShedRecord` (fields `Tag`, `Older`, `StartedAt` and `Entries`); new file `runtime.go`.
`RunTime` holds `Span` (an `Interval`), `Length` (the span's duration), `StatusRead` (a shed status was read), `Roles` (a duration per role key), `Idle`, `Producers` (a slice of `ProducerTime`: `Producer`, `Entries`, `Elapsed`, `Idle`) and `ProducersSpan` (`Length` when `StatusRead`, else zero, the denominator of the Producers shares).
`RunTally` gains `Time RunTime`; `CountRun` ends by calling `run.SetTime(nil)`, so a run counted without a status carries its transcript-window time and every existing caller of `CountRun` gets a populated `Time`.
`spanOf` with a nil record, or a record with no history entry, returns `[First, Last]`, the transcript window.
With a record the end is the latest `End` among its entries, and the start is the record's `StartedAt`, or `run.First` when that is zero, then raised to `Older` when earlier than it, and finally capped at the first entry's `End` (a zero start, from a seed without `started_at` and a run with no timestamped transcript line, takes the first entry's `End`); the cap keeps the start at or before every entry end, so the span is never negative.
`SetTime` sets `StatusRead` to whether the record supplied the span, then sets `Roles[role]` to `totalOf(clipTo(role.Work, span))` for every role of the run, `Idle` to `Length` minus `totalOf(clipTo(run.Work, span))`, and `Producers` only when `StatusRead`.
`run.Work` is built from the run's own project directory alone, so the orch session and every orch sub-agent, including one charged wholly to the run for tokens, stay out of the union and parent review counts as idle.
`producerTimes` splits the span by entry in recorded order: each entry owns the interval from the latest end among the entries before it (the span start for the first) to its own end, empty when its end is not later.
Per producer name it sums `Entries` (every entry, empty interval or not), `Elapsed` (the summed intervals) and `Idle` (each interval's length minus the part `clipTo(work, interval)` covers); the rows are sorted by `Elapsed` descending, then by name, and `Elapsed` sums to `Length`.
`Publish` and `Finalize` carry no inner breakdown, so their whole interval is their row.
`runtime_test.go` holds `TestRunTime`, one table; each row builds a run with `writeLines` transcripts and `CountRun`, an optional `ShedRecord`, and asserts the `RunTime` after `SetTime`.
Rows: with a seed start, the span runs from `StartedAt` to the latest entry end, idle is the span minus the union of all working intervals and a role's time is clipped to the span, with a session that started before the span start contributing only its part inside it; a record with no `StartedAt` starts at the earliest transcript line; a record with no `StartedAt` and a run with no timestamped line starts at the first entry's end; a start earlier than `Older` is raised to it; a start after the first entry's end is capped at it; no record uses the transcript window, leaves `Producers` empty and `StatusRead` false; a record with an empty history behaves as no record; entries out of order (the second ending earlier than the first) give `Elapsed` that sums to `Length`, with the early entry's interval empty but counted in `Entries`; two entries of one producer sum into one row; a producer's `Idle` is the part of its intervals no transcript works in; a prime project directory holds an orch session and an orch sub-agent whose first prompt names the run, both working inside the span while the run's own transcripts do not, and `CountOrch` runs over the run; `Idle` is then the span minus the run's own union alone, which pins that the run's `Work` takes its lines from the run's project directory only.
A folded entry's `last_at` end is resolved in Card 3's `ReadShedRecord`, so the hand-built records here carry `End` values and need no fold row.
The tokencount row in `test-budget.yaml` rises by one.

**ImpactSummary:** `CountRun` has callers in `run` and several tests that gain a populated `Time` and read nothing from it yet, so no existing assertion changes; the new fields and functions are additive.
