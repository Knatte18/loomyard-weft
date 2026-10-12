# Card 4 — tokencount-run-times

**Create:**
- `plan:tools/tokencount#RunTimes` -> `type RunTimes struct`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#RunTally.Times` -> `func (run RunTally) Times() RunTimes`
- `plan:tools/tokencount#spanOf` -> `func spanOf(run RunTally) Interval`
- `plan:tools/tokencount#clipIntervals` -> `func clipIntervals(in []Interval, span Interval) []Interval`
- `plan:tools/tokencount#intervalsTotal` -> `func intervalsTotal(in []Interval) time.Duration`
- `plan:tools/tokencount#TestRunTimes` -> `func TestRunTimes(t *testing.T)`

**Edit:**
- `tools/tokencount#RunTally`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** A run's span, role times, idle and Producers rows are computed from what cards 2 and 3 collected (Decisions 2 to 6); nothing prints them yet.
`RunTally` gains `Shed *ShedRecord`, nil for a run whose archived status was not read, and the new file `tools/tokencount/times.go` declares the rest.
`RunTimes` holds `Span Interval`, `Idle time.Duration`, `Roles map[string]time.Duration` (every `Roles` key, `+sub` rows included), `Producers []ProducerTime` and `NoStatus bool`; `ProducerTime` holds `Name`, `Entries`, `Elapsed` and `Idle`.
`spanOf` returns the transcript window `[First, Last]` when `Shed` is nil.
With a record, the end is the latest `End` among the `Entries`, and the start is `Shed.Start`, else `First`; a start later than the first entry's `End` becomes that `End`, a start earlier than `Shed.Older` becomes `Older` when `Older` is set, and a zero start (no stamp and no transcript line) becomes the first entry's `End`.
The lower bound is applied before the upper, so the first entry's end always wins and the start never passes it.
`clipIntervals` cuts a sorted, disjoint list to a span and `intervalsTotal` sums one.
`Times` clips each role's `Work` to the span for `Roles`, sets `Idle` to the span's length minus the clipped length of the run's `Work` union, and builds `Producers` from the entries in recorded order: entry i owns `[latest End among entries before it (the span start for the first), its End]`, and owns nothing when its End is not later; `Elapsed` sums those intervals per producer name, `Entries` counts every entry of the name, `Idle` is the part of the intervals the run's `Work` does not cover, and the rows are ordered by `Elapsed` descending, then name.
Elapsed therefore sums to the span, entries out of order included.
`Producers` is nil and `NoStatus` true when `Shed` is nil, and the shares are left to the printing card.
`TestRunTimes` is one table over hand-built `RunTally` values with `Work` lists in minutes from a fixed base; rows:
- an archived status with four entries, the third ending before the second so it owns nothing, and role work partly outside the span, asserting the span from the seed start to the latest end, every role's clipped time, the idle, and each producer's elapsed, idle and entry count with the elapsed summing to the span;
- a start of zero taking `First`, a start below `Older` taking `Older`, a start after the first entry's end taking that end, an `Older` later than the first entry's end also yielding that end (the upper bound wins), and no start and no `First` taking the first entry's end;
- no `Shed`, asserting the window span, `NoStatus`, no producers and an idle that is the window minus the work in it.

`cmd/lyx/testdata/test-budget.yaml` raises the `tools/tokencount` row by one for `TestRunTimes`.

**ImpactSummary:** `RunTally` only gains a field, so the package's composite literals compile unchanged and no existing output differs.
