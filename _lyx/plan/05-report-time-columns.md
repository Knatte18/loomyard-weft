# Card 5 — report-time-columns

**Edit:**
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, timing RunTiming)`
- `plan:tools/tokencount#TestRunTiming`

**Create:**
- `plan:tools/tokencount#RunTiming.add` -> `func (t *RunTiming) add(o RunTiming)`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, rows []ProducerRow, span time.Duration)`
- `plan:tools/tokencount#timeRule` -> `const timeRule = "Time is turn time: a session works from a user or assistant line until the next line, unless that line ends the turn; idle is the part of the span in which no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push. Role time shares can sum past 100% where sessions overlap, a +sub row overlapping its parent; the union of working time plus idle is the span."`
- `plan:tools/tokencount#producerRule` -> `const producerRule = "A row's elapsed is the gap from the previous row's finish time to its own, so it includes any wait before the row started; elapsed sums to the span."`

**Uses:**
- `plan:tools/tokencount#RunTiming`
- `plan:tools/tokencount#ProducerRow`

**Intent:** Print the time, Decisions 4, 5 and 7.
`writeTable` takes the table's `RunTiming` and gains two columns after `share`, `time` and `time share`: a role's `Roles` duration rounded to the second and printed as a `time.Duration`, and that duration over the span's length as a percentage; `share` keeps its weight meaning and the rows keep their weight order.
Below the role rows and above the orch row it prints one `idle` row with `Idle` and its share and every token cell blank; the orch row's time cells are blank.
`writeProducers` prints a "Producers" table of `producer`, `entries`, `elapsed`, `idle` and `share`, share being `Elapsed` over `span`, followed by `producerRule`.
`RunTiming.add` sums a run's timing into the "All runs" timing: each role's duration, `Idle`, the span's length as a plain sum of lengths, and the Producers rows merged by name with entries, elapsed and idle summed; no union is taken across runs, and the merged rows are reordered by `Elapsed` descending, then name.
`Report.WriteMarkdown` folds every run's `Timing` into one `RunTiming` for the "All runs" table, which prints `writeProducers` when any run has status, then `timeRule` once after the "All runs" table.
Each run's section prints its table with its own timing; when `HasStatus` it prints its Producers table, else one line of `NoStatus` so the reader sees idle covers the transcript window only.
The "All runs" span used for "time share" is the sum of the runs' spans, as `add` builds it.
No test asserts the old header line or the title, so no existing assertion changes; `TestCountRun`'s row assertion is a prefix that still holds.
`TestRunTiming` gains a rendering block: a `Report` of two runs, one with status and one without, written to a buffer, asserting the exact header line, one role row with its `time` and `time share` cells, the `idle` row, the orch row's blank time cells, the Producers table of the run with status and the "All runs" Producers table summing both, the `NoStatus` line of the run without status, and `timeRule` present once.

**ImpactSummary:** `writeTable`'s one caller, `WriteMarkdown`, is on this card; the report's header line and every per-run section change shape, and no test outside this package reads them.
