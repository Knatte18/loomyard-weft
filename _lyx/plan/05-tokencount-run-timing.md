# Card 5 — tokencount-run-timing

**Create:**
- `plan:tools/tokencount#RunTiming` -> `type RunTiming struct`
- `plan:tools/tokencount#ProducerTiming` -> `type ProducerTiming struct`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(entry shedengine.HistoryEntry) (time.Time, error)`
- `plan:tools/tokencount#timeRun` -> `func timeRun(run RunTally, records *ShedRecords) (RunTiming, error)`
- `plan:tools/tokencount#TestRunTiming` -> `func TestRunTiming(t *testing.T)`

**Edit:**
- `tools/tokencount#RunTally`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#Interval`
- `plan:tools/tokencount#unionIntervals`
- `plan:tools/tokencount#clipIntervals`
- `plan:tools/tokencount#totalDuration`
- `plan:tools/tokencount#ShedRecords`
- `internal/shedengine#HistoryEntry`

**Intent:** Turn a run's working intervals and its archived records into the figures the report prints, per Decisions 3, 4 and 5 of the decision record.
`RunTiming` holds `Span Interval`, `HasStatus bool`, `Roles map[string]time.Duration`, `Idle time.Duration` and `Producers []ProducerTiming`;
`ProducerTiming` holds `Name`, `Entries int`, `Elapsed` and `Idle time.Duration`.
`RunTally` gains a `Timing RunTiming` field, which card 7 fills.
`entryEnd(entry)` parses the entry's `LastAt` when `Repeats` is over zero, else its `At`, as RFC 3339, and returns the parse error otherwise.
`timeRun(run, records)` computes the span first.
With nil records or a status whose `History` is empty, the span is the transcript window `run.First` to `run.Last`, `HasStatus` is false and `Producers` is nil.
Otherwise `HasStatus` is true and the span ends at the latest `entryEnd` over the history;
it starts at the seed's `StartedAt` when set, else at `run.First`, and either start is raised to `records.PriorTagAt` when that is later and lowered to the first history entry's end when that is earlier.
`Roles` maps each key of `run.Working` to `totalDuration(clipIntervals(intervals, span))`.
`Idle` is the span's length minus the total of the union of every `run.Working` set clipped to the span;
orch transcripts never enter `run.Working`, so parent review is idle by construction.
`Producers` walks the history in recorded order: each entry owns the interval from the latest `entryEnd` among the entries before it, the span start for the first entry, to its own end, empty when its own end is not later;
per producer name, in order of first appearance, `Entries` counts entries, `Elapsed` sums the intervals, and `Idle` sums each interval's length minus the total of the run's working union clipped to it.
Elapsed over every producer therefore sums to the span.
An unparseable `StartedAt`, `At` or `LastAt` is an error naming the run's slug and the field, and `run` aborts on it.
The types and functions go in `timing.go`.
`TestRunTiming` in `timing_test.go` builds `RunTally` values with hand-written `Working` sets and `First`/`Last` windows beside `ShedRecords` values, calls `timeRun`, and asserts `RunTiming` exactly, as a table: a seed with `started_at` and a history with a folded entry spans from `started_at` to the folded entry's `last_at`, each role's time is its clipped union and idle is the span minus the union of all roles;
a seed without `started_at` starts at the earliest transcript line, raised to `PriorTagAt` when the transcripts begin earlier, and lowered to the first entry's end when the transcripts begin later;
history entries recorded out of time order still give elapsed figures that sum to the span, and a producer's idle is the part of its intervals no working interval covers;
two producers sharing a name fold into one row with two entries;
nil records give the transcript window, `HasStatus` false and no producers;
and an entry with an unparseable `at` is an error naming the slug.
Raise the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 14 to 15 for the new test.

**ImpactSummary:** Adds a Timing field to RunTally and pure computation over it; no caller fills or prints it until cards 6 and 7.
