# Card 5 — time by shed producer

**Create:**
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#producerTimes` -> `func producerTimes(history []shedengine.HistoryEntry, window Interval, busy []Interval) []ProducerTime`

**Edit:**
- `plan:tools/tokencount#RunTime`
- `plan:tools/tokencount#RunTally.Measure`
- `plan:tools/tokencount#TestRunTimeMeasure`

**Uses:**
- `plan:tools/tokencount#spanOf`
- `plan:tools/tokencount#entryEnd`
- `plan:tools/tokencount#clipIntervals`
- `plan:tools/tokencount#totalOf`

**Intent:** The record's Producers table answers "the time each producer takes" from the run's shed history, and this card computes its rows; no report prints them yet.
`producerTimes` goes in `tools/tokencount/worktime.go`.

`ProducerTime` holds `Producer string`, `Entries int`, `Elapsed time.Duration` and `Idle time.Duration`.
`producerTimes` walks `history` in recorded order with a running latest end that starts at `window.Start`.
An entry with a parseable `entryEnd` owns the interval from the running end to its own end, empty when its own end is not later, and then raises the running end; an entry with no parseable end is not counted and owns nothing.
Per producer name it sums the entries counted, the lengths of the owned intervals (`Elapsed`) and, for `Idle`, each owned interval's length minus the total of `busy` clipped to that interval, the part of it no working interval of the run covers.
Because the intervals tile from `window.Start` to the latest end, the rows' `Elapsed` sums to the span, an entry recorded out of order included.
Rows are ordered by `Elapsed` descending, then by name, so the slowest producer is first.
The report states, in a later card, that a row's elapsed is the gap between finish times and so includes any wait before the row started.

`RunTime` gains `Producers []ProducerTime`, nil when no shed status was read, and `ProducerSpan time.Duration`, the span the producer shares divide by: the run's span when its status was read, else zero.
`Measure` passes `producerTimes` the status history, the span and the union of every role's clipped `Work`, which it already builds for `Idle`, and sets the two fields only when `spanOf` reported a status read.
No start timestamp is added to the shed history entries; `shedengine`'s status shape does not change.
`Publish` and `Finalize` record no inner breakdown, so each appears as one row holding its whole interval.

`TestRunTimeMeasure` gains a `wantProducers []ProducerTime` field, nil for the rows without a status, and each status row asserts it, with `Elapsed` summing to `Span`.
The rows that carry producers: the status whose last entry ends after the transcripts, where the first producer's interval starts at the seed's `started_at` and its `Idle` is the part no role works in; the folded entry, whose interval ends at its `last_at` and which counts as one entry; entries recorded out of order, where the entry ending before the running end counts and owns nothing and the later entries tile on from the running end; the unparseable-`at` entry, which is not counted.
This card adds two rows of its own: the same producer name on two entries, where entries and elapsed sum, and two producers with equal elapsed, ordered by name.
The empty-owned-interval branch is reached by the out-of-order row, whose second entry ends before the first.

No new top-level test is added: the rows extend `TestRunTimeMeasure`, which already builds the runs and archives these rows need.

**ImpactSummary:** `RunTime` gains two fields and `Measure` one call; the only reader of either is the report card after this one, and no existing assertion reads `RunTime`.
