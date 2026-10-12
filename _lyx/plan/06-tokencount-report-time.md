# Card 6 — tokencount-report-time

**Create:**
- `plan:tools/tokencount#formatDuration` -> `func formatDuration(d time.Duration) string`
- `plan:tools/tokencount#sumTimings` -> `func sumTimings(runs []RunTally) RunTiming`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, producers []ProducerTiming, span time.Duration)`

**Edit:**
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, timing RunTiming)`
- `tools/tokencount#TestCountOrch`
- `plan:tools/tokencount#TestRunTiming`

**Uses:**
- `plan:tools/tokencount#RunTiming`
- `plan:tools/tokencount#ProducerTiming`
- `tools/tokencount#orchRole`

**Intent:** Print the time figures in the role tables and a Producers table per run and over all runs, per Decisions 4, 5 and 7 of the decision record.
`formatDuration` prints a duration rounded to the second with `time.Duration`'s `String`.
`writeTable` takes the table's `RunTiming` and writes two new columns after `share`, `time` and `time share`, the role's duration and its percentage of the span, keeping the weight order of the rows and the meaning of `share`;
below the role rows it writes an `idle` row whose token, weight, share, cost and model cells are blank and whose time cells hold `timing.Idle` and its share;
the orch row keeps its place after the roles and leaves both time cells blank.
A time share divides by the span's length and prints `n/a` when the span is zero.
`sumTimings(runs)` adds each run's `Timing` into one `RunTiming` for the "All runs" table: the span's length as a sum of the runs' span lengths, kept as an `Interval` from zero to that sum, each role's time summed by role, idle summed, and the producers summed by name over the runs whose `HasStatus` is true, with no union taken across runs.
`writeProducers` writes a table with the columns `producer`, `entries`, `elapsed`, `idle` and `share`, one row per `ProducerTiming`, share being elapsed over `span`.
`Report.WriteMarkdown` passes `sumTimings(r.Runs)` to the "All runs" table and each run's `Timing` to its own, writes the Producers table after each role table whose timing has `HasStatus`, and after a run's table whose timing lacks it writes a line saying no shed status was read for the run, so idle covers only the transcript window and there is no Producers table.
Below the "All runs" table it writes the definitions in prose: time is the union of a role's sessions' turns, from a user or assistant line to the next while the turn is open, clipped to the run's span;
idle is the time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push;
the span runs from the seed's start to the last shed row's finish, or over the transcript window when no status was read;
role shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent, and the union of working time plus idle is the span;
a Producers row's elapsed is the gap between finish times, so it includes any wait before the row started;
and "All runs" sums each run's times, idle and span.
Update the two exact table rows `TestCountOrch` asserts to carry the new columns, with the orch row's time cells blank and the `idle` row present.
Extend `TestRunTiming` with a report step: set `Timing` on two `RunTally` values, one with status and one without, write the `Report` and assert the exact `idle` row and the `time` and `time share` cells of one role in each run's table, the Producers table rows of the run with status, the no-status line of the other, and the "All runs" `idle` row and Producers rows as the sums of the two.

**ImpactSummary:** Every role table gains two columns and an idle row and each run section may gain a Producers table, so the exact-row assertions in TestCountOrch change; TestCountRun and TestEstimatedCost match on row prefixes and suffixes and keep passing.
