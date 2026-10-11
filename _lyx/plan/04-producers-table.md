# Card 4 — producers-table

**Create:**
- `tools/tokencount/producers.go`
- `plan:tools/tokencount#ProducerRow` -> `type ProducerRow struct`
- `plan:tools/tokencount#producerRows` -> `func producerRows(entries []producerEntry, span interval, working []interval) []ProducerRow`
- `plan:tools/tokencount#sumProducerRows` -> `func sumProducerRows(perRun [][]ProducerRow) []ProducerRow`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, rows []ProducerRow, span time.Duration)`
- `tools/tokencount/producers_test.go`
- `plan:tools/tokencount#TestProducerRows` -> `func TestProducerRows(t *testing.T)`

**Edit:**
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#producerEntry`
- `plan:tools/tokencount#interval`
- `plan:tools/tokencount#coveredWithin`

**Intent:**
Split each run's span by shed history entry into a per-run "Producers" table and its sum over runs in "All runs" (decision record, Decisions 5 and 6), so the slowest producer and the row that holds the idle time stand out.

`ProducerRow` is one producer name's `Producer`, `Entries` count, `Elapsed` and `Idle` durations.
`producerRows`, in `producers.go`, walks `entries` in recorded order with a running latest end that starts at `span.From`: each entry owns the interval from that running end to its own end, empty when its own end is not later, and the running end then moves to the later of the two.
Per producer name it counts the entries, sums the owned intervals as `Elapsed`, and sums as `Idle` each owned interval's length minus `coveredWithin(working, from, to)` of it.
Card 3 never lets the start pass the first entry's end and ends the span at the latest entry end, so the rows' `Elapsed` sums to the span, entries recorded out of time order included.
Rows are ordered by `Elapsed` descending, then by name.
`sumProducerRows` adds the rows of several runs per producer name, in the same order.
`writeProducers` writes a `### Producers` table, `| producer | entries | elapsed | idle | share |`, durations rounded to the second and share the elapsed as a percentage of `span`, followed by the sentence that a row's elapsed is the gap between finish times, so it includes any wait before the row started; `Publish` and `Finalize` record no inner breakdown, so their whole interval is their row.

`Report.WriteMarkdown` writes each run's table after its role table, from its `Entries`, `Span` and `Working`, and none for a run with `NoStatus` set.
"All runs" writes the `sumProducerRows` of the runs with a table, its share over the sum of those runs' spans, and states how many of the counted runs it covers; with none it writes no Producers table.

The package doc in `main.go` gains a paragraph on the Producers table: how a history entry owns its interval, the columns, that elapsed sums to the span, and the "All runs" sum.
The description comments of `tokencount.sh` and `tokencount.cmd` say the report holds token usage and time by role, an `idle` row per role table and a Producers table per run; the output path is unchanged.

`TestProducerRows`, new in `producers_test.go`, is a unit-tier table asserting exact rows:
three entries of two producers, one recorded with an end earlier than its predecessor's, own intervals whose `Elapsed` sums to the span, the out-of-order entry counted with zero elapsed;
a working interval inside one entry's interval reduces that producer's `Idle` by exactly its overlap;
`sumProducerRows` over two runs adds entries, elapsed and idle per producer;
a report over one run with entries and one with `NoStatus` writes the first run's table, no table for the second, and an "All runs" table whose share divides by the first run's span only.
The new top-level test pushes `tools/tokencount` past its `test-budget.yaml` row, so the row rises to the package's new count.

**ImpactSummary:** Adds a Producers table to each run section and to "All runs" of the `tools/tokencount` report, and rewords the package doc and the two launchers' description comments; no existing table changes.

**Commit:** 4: tokencount: Producers table from the shed history
