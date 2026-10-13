# Card 4 — Producers table

**Create:**
- `plan:tools/tokencount#ProducerRow` -> `type ProducerRow struct`
- `plan:tools/tokencount#RunTally.Producers` -> `func (run RunTally) Producers() []ProducerRow`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, rows []ProducerRow, span time.Duration)`

**Edit:**
- `tools/tokencount#Report.WriteMarkdown`
- `plan:tools/tokencount#TestReadArchive`

**Intent:**
Decisions 5 and 6: a run read with shed status gets a "Producers" table that splits its span by history entry, in the shed's own row names; no start timestamp is added to `shedengine`'s history entries, and `Publish` and `Finalize` show as their whole interval.

`ProducerRow` is one producer name's `Producer`, `Entries`, `Elapsed` and `Idle`.
`Producers`, in `archive.go`, returns nil for a run without `HasStatus`.
Otherwise it walks `History` in recorded order: each entry owns the interval from the latest end among the entries before it, or `SpanFrom` for the first entry, to its own `entryEnd`, and the interval is empty when its own end is not later.
Entry ends were parsed when card 3's `readArchive` read the status, so an end here always parses.
Per producer name, in order of first appearance: the entry count, the summed interval lengths as `Elapsed`, and as `Idle` the part of those intervals that no working interval of the run covers, `unionLength` of all the run's work clipped to each interval, subtracted from it.
Since the span starts at or before the first entry's end and ends at the latest end, `Elapsed` sums to the span, entries out of order included.

`writeProducers` writes `| producer | entries | elapsed | idle | share |`, durations through `formatDuration` and share as `%.1f%%` of `span`.
`Report.WriteMarkdown` writes, after each run's span line, `### Producers` and the run's table, nothing for a run without status, followed by one line: a row's elapsed is the gap between finish times, so it includes any wait before the row started.
"All runs" gains the same table below its statements, the rows summed by producer name over the runs with status, in order of first appearance across runs, share over the sum of those runs' spans; it is omitted when no run has status.

Tests.
`TestReadArchive` gains rows that call `Producers` on the read run:
- a history whose second entry ends before the first (out of order) gives the second an empty interval, and the rows' `Elapsed` sum to `Span`;
- a producer whose interval holds a transcript gap reports that gap as its `Idle`, and one fully covered by a turn reports zero;
- two entries of one producer name sum into one row with `Entries` 2;
and a row renders `Report.WriteMarkdown` over a run with status and one without, asserting the run's `### Producers` table rows, the elapsed note, no `### Producers` under the run without status, and the "All runs" table's summed rows.

**ImpactSummary:** confined to `tools/tokencount`'s report: a run with archived status gains a Producers section, and "All runs" gains the summed table; no existing test asserts the absence of either.

**Commit:** 4: tokencount: the Producers table of a run's shed rows
