# Card 4 — The Producers table

**Create:**
- `plan:tools/tokencount#ProducerRow` -> `type ProducerRow struct`
- `plan:tools/tokencount#RunTally.Producers` -> `func (run RunTally) Producers() []ProducerRow`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, rows []ProducerRow, span time.Duration)`

**Edit:**
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`
- `plan:tools/tokencount#TestReadArchive`
- `plan:tools/tokencount#TestRunReadsArchivedStatus`

**Uses:**
- `plan:tools/tokencount#ArchivedRun`
- `plan:tools/tokencount#coveredWithin`
- `plan:tools/tokencount#RunTally.ReadArchive`
- `plan:tools/tokencount#fakeArchive`

**Intent:**
Split each run's span by shed history entry so the slowest producer and the row that holds the idle time stand out (Decision 5).
`Publish` and `Finalize` record no inner breakdown, so their whole interval is their row (Decision 6).

In `archive.go`:
- `ProducerRow` holds a producer name's `Producer`, `Entries`, `Elapsed` and `Idle`.
- `Producers()` returns nil when `Archive` is nil.
  Otherwise it walks `Archive.Entries` in recorded order, keeping the latest end so far, which starts at `SpanStart`: each entry owns the interval from that latest end to its own end, empty when its own end is not later, and then the latest end becomes the later of the two.
  Per producer, in order of first appearance, `Entries` counts its entries, `Elapsed` sums its intervals, and `Idle` sums each interval minus `coveredWithin` of the run-wide working union card 2 keeps over it.
  Because the first interval starts at the span start, which card 3 caps at the first entry's end, and the walk ends at the latest entry end, the elapsed column sums to the span, entries out of order included.
- `writeProducers` writes `### Producers` and `| producer | entries | elapsed | idle | share |`, one row per producer with `formatDuration` for the durations and share `100 * Elapsed / span` printed `%.1f%%`, `0.0%` when the span is 0.

`Report.WriteMarkdown`:
- Each run with an `Archive` gets its Producers table below its role table; a run without one gets none, beside card 3's note.
- "All runs" gets, after its notes and before the Orchestrator section, a Producers table summing `Entries`, `Elapsed` and `Idle` per producer over the runs with an `Archive`, in first-appearance order across runs in report order, with share over the sum of those runs' spans; no table when no run has one.
  Below it, once: a row's elapsed is the gap between finish times, from the latest earlier entry's finish to its own, so it includes any wait before the row started; its idle is the part of that gap in which no session of the run is in a turn; and runs with no shed status are left out of this table.

Package doc in `main.go` (the file holds no member this card changes): a paragraph after card 3's span paragraph states Decision 5's split, the columns, the "All runs" sum and its denominator, and that a row's elapsed includes any wait before the row started.

Launchers: `tokencount.sh` and `tokencount.cmd` describe the report as token usage and time by role: each role's working time and time share beside its tokens, measured as turn time, an idle row of time no session of the run is in a turn, and a Producers table splitting the run's span by shed history entry; each points at `tools/tokencount/main.go`'s package doc for how each is measured.

Tests:
- `TestReadArchive` gains a `wantProducers []ProducerRow` column, checked against `Producers()` on every row that wants no error (nil where `Archive` is nil), and a row over its usual transcript with a seed `started_at` 10:00 and a history, in order, `A` ending 10:20, `B` ending 11:00, `C` ending 10:40 (earlier than the latest end so far), and `B` folded (`at` 11:05, `repeats` 1, `last_at` 11:20).
  That row wants `A` 1 entry 20m elapsed 0s idle, `B` 2 entries 1h elapsed 40m idle, `C` 1 entry 0s elapsed, elapsed summing to the 1h20m span.
  After the rows, the test renders a `Report` of that row's run under two slugs and asserts the per-run row `| B | 2 | 1h0m0s | 40m0s | 75.0% |`, the "All runs" row `| B | 4 | 2h0m0s | 1h20m0s | 75.0% |` and the sentence that a row's elapsed includes any wait before the row started.
- `TestRunReadsArchivedStatus` asserts alpha's section carries `### Producers` with a row per producer of its fixture history, and beta's section none.
- No new top-level test, so the `tools/tokencount` budget row stays at card 3's 16.

**ImpactSummary:** confined to `tools/tokencount`'s report writer, its package doc and its two launchers' comments; `Report.WriteMarkdown` keeps its signature, and runs without an archived status render as before.

**Commit:** 4: tokencount: the Producers table
