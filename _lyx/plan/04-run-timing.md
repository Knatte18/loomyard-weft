# Card 4 — run timing

**Create:**
- `tools/tokencount/timing_test.go`
- `plan:tools/tokencount#RunTiming` -> `type RunTiming struct`
- `plan:tools/tokencount#ProducerRow` -> `type ProducerRow struct`
- `plan:tools/tokencount#RunTally.Timing` -> `func (run RunTally) Timing(shed *ShedRecord) RunTiming`
- `plan:tools/tokencount#timingSpan` -> `func timingSpan(run RunTally, shed *ShedRecord) Interval`
- `plan:tools/tokencount#producerRows` -> `func producerRows(span Interval, shed *ShedRecord, worked []Interval) []ProducerRow`
- `plan:tools/tokencount#TestRunTiming` -> `func TestRunTiming(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** `RunTally.Timing` turns a run's working intervals and its optional `ShedRecord` into the run's span, each role's time, idle and the Producers rows, in a new file `tools/tokencount/timing.go`.
`RunTiming` holds `Span Interval`, `Length time.Duration` (the span's length), `ShedRead bool`, `Roles map[string]time.Duration`, `Idle time.Duration`, and `Producers []ProducerRow` in first-appearance order; `ProducerRow` holds the producer name, `Entries int`, `Elapsed` and `Idle` durations.
`timingSpan` ends the span at the latest entry end, and starts it at `ShedRecord.Started`, or at `run.First` when that is zero; either start is then raised to `Floor` when `Floor` is set, and lowered to the first entry's end when it lies later.
With a nil record the span is `run.First` to `run.Last`, and `ShedRead` is false.
Each role's time is `intervalsTotal` of `unionIntervals(run.Work[role])` clipped to the span, `+sub` roles included, so concurrent sessions of one role count once.
`Idle` is the span length minus the total of the clipped union of every role's intervals in `run.Work`; the orch session lives in the prime's project directory, never in `run.Work`, so it and its sub-agents are in no union and parent review counts as idle.
`producerRows` walks the entries in recorded order, giving each the interval from the running latest end of the entries before it (the span start for the first) to its own end, empty when its end is not later; per producer name it sums entries, the interval lengths as `Elapsed`, and as `Idle` the part of those intervals that the clipped union of all working intervals does not cover.
The elapsed times sum to the span length however the entries are ordered, and a nil record gives no rows.
This adds no verb, flag or guard: it only reads the intervals and the record, and the bounds of Decision 3 keep a reused slug's older sessions out of the span.
`TestRunTiming` is one new top-level test, a table of runs built with `CountRun` over synthetic transcripts, and its rows are:
- a seed with `started_at` and a folded final entry: the span runs from the seed to the entry's `last_at`, and `Idle` is the span minus the union of the working intervals;
- entries out of order: `Elapsed` sums to the span, and each producer's `Idle` is the part of its intervals no transcript works in;
- a seed without `started_at`: the span starts at the earliest transcript line when `Floor` is zero and the line is before the first entry's end; a transcript line earlier than `Floor` raises the start to `Floor`; a line later than the first entry's end lowers the start to that end;
- a seed with `started_at` earlier than `Floor` is raised to `Floor`, and one later than the first entry's end is lowered to it;
- transcript lines before the span start and after the span end are clipped: a role's time and `Idle` count only the part inside the span;
- a nil record: the span is the transcript window, `ShedRead` is false and there are no rows;
- a `+sub` role keeps its own entry in `Roles`, apart from its parent's.
The orch exclusion holds by construction, since `CountRun` reads only the worktree's project directory, and card 5's report test pins it on the printed `idle` row.
The test sits in the new file `tools/tokencount/timing_test.go`, runs with `t.Parallel()`, and reuses `stampedAssistant`, `stampedUser` and the `ShedRecord` literal; the new top-level test takes the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 15 to 16.

**ImpactSummary:** The budget file's `tools/tokencount` row is the only existing text this card edits; `Timing`, its types and helpers are new and have no caller until card 5.
