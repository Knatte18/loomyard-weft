# Card 5 — time report

**Create:**
- `plan:tools/tokencount#sumTimes` -> `func sumTimes(runs []RunTally) RunTime`
- `plan:tools/tokencount#writeTimeNotes` -> `func writeTimeNotes(w io.Writer, t RunTime)`
- `plan:tools/tokencount#writeTimeDefinitions` -> `func writeTimeDefinitions(w io.Writer)`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, t RunTime)`
- `tools/tokencount/timereport_test.go`

**Edit:**
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, times RunTime)`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#TestCountOrch`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#RunTime`
- `plan:tools/tokencount#ProducerTime`
- `tools/tokencount#RunTally`
- `tools/tokencount#TestCountRun`

**Intent:**
Decision 7 and the report half of Decisions 3, 4 and 5: the computed times reach the markdown; new file `timereport.go`.
`writeTable` gains the `times` parameter and its header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`.
A role row's `time` is `times.Roles[role]` rounded to the second and printed as a `time.Duration`; its `time share` is that over `times.Length` as `%.1f%%`, `0.0%` when the length is zero.
`share` keeps its weight meaning and rows keep their weight order; the `orch` row prints blank `time` and `time share` cells, because orch spans many runs; after the role rows and the orch row the table ends with `| idle | | | | | | | | <idle> | <share> | | |`, with blank token cells.
`Report.WriteMarkdown` passes `sumTimes(r.Runs)` to the All-runs table and `run.Time` to each run's table.
`sumTimes` adds each run's `Length`, each role's time, `Idle` and `ProducersSpan`, and merges the `Producers` rows by name (entries, elapsed and idle summed, sorted as in Card 4), taking them only from runs with `StatusRead`; no union is taken across runs, and `StatusRead` is true when any run's is.
After each role table `writeTimeNotes` prints one line: for a run with `StatusRead`, its span as start, end and duration, and that it runs from the seed's `started_at`, or the earliest transcript line when the seed has none, to the latest shed history entry end; for a run without, that no shed status was read, so the span is the transcript window and idle covers only that window.
`writeTimeDefinitions` prints once, in the All-runs section after the table: that role shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent; that the union of working time plus idle is the span; that working time is turn time, an interval between consecutive user or assistant lines unless the first ends the turn; and that idle is the time no session of the run is in a turn, holding parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push.
`writeProducers` prints, for a `RunTime` whose `StatusRead` is true, a `### Producers` heading and the table `| producer | entries | elapsed | idle | share |`, share being elapsed over `ProducersSpan`, with the sentence that a row's elapsed is the gap between finish times, so it includes any wait before the row started; it prints nothing otherwise, which is how a run without a status gets no Producers table.
The per-run section calls it after its notes, and the All-runs section calls it with the summed `RunTime`.
`TestCountOrch` asserts exact role and orch rows that gain the two columns, so its expected strings change: each burler row gains its time and time share before the cost (the fixture's one user line and one assistant line per run make each run's burler time its whole window, so `| 4h0m0s | 100.0% |` in the All-runs row and `| 2h0m0s | 100.0% |` in run-a's row), and each orch row gains two blank cells between its share and its cost; `TestCountRun` and `TestEstimatedCost` assert prefixes and cells that stay as they are.
`timereport_test.go` holds `TestReportTimeColumns`, which builds `RunTally` values with a hand-set `Time` and asserts the output of `Report.WriteMarkdown`: the exact header, which `TestCountOrch` does not pin; the `idle` row's cells (the role rows' time cells and the orch row's blank cells are pinned by `TestCountOrch`); All runs summing two runs' role times, idle and spans; a run with a status printing the span line and a Producers table whose elapsed cells sum to the span, and the All-runs Producers table summing the two runs' rows with its shares over the summed span of the runs that have a status; a run without a status printing the no-shed-status line and no `### Producers` heading under its own section; the definitions text containing `can sum past 100%`, `no session of the run is in a turn`, `plus idle is the span` and a statement that time shares divide by the table's span, and the Producers section containing `the gap between finish times`.
The tokencount row in `test-budget.yaml` rises by one.

**ImpactSummary:** `writeTable` has one caller, `Report.WriteMarkdown`; `TestCountOrch` is the one test whose exact row assertions break, and the other report assertions match on prefixes before the new columns.
