# Card 4 — time-columns

**Create:**
- `plan:tools/tokencount#timeRule` -> `const timeRule = ""`
- `plan:tools/tokencount#durationCell` -> `func durationCell(d time.Duration) string`
- `plan:tools/tokencount#timeShareCell` -> `func timeShareCell(d, span time.Duration) string`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, producers []ProducerTime, span time.Duration)`
- `plan:tools/tokencount#sumProducers` -> `func sumProducers(runs []RunTally) []ProducerTime`

**Edit:**
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, idle, span time.Duration)`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#TestCountOrch`
- `plan:tools/tokencount#TestArchivedRunTiming`

**Intent:**
Print the times card 3 computes (Decisions 4, 5 and 7).

- `durationCell` prints `d.Round(time.Second).String()`.
  `timeShareCell` prints `d` over `span` as `%.1f%%`, or `n/a` when the span is not positive.
- `writeTable` gains `time` and `time share` columns after `share`: `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`.
  Rows keep their weight order, and `share` keeps its meaning.
  The orch row's two time cells are blank.
  After the role rows and the orch row comes an `idle` row with blank token, weight, share, cost and models cells and its time and time share.
- `Report.WriteMarkdown`'s "All runs" table sums each run's role `Time`, `Idle` and span (`End - Start`) and passes the sums, so its time shares divide by the sum of the spans; no union is taken across runs.
  Below it, when any run has `Producers`, it writes `sumProducers` (summed per producer name in first-appearance order across the runs) as a "Producers" table over the summed spans of those runs.
  It then writes `timeRule` once: idle is the time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push; role time shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent; the union of working time plus idle is the span; and a Producers row's elapsed is the gap between finish times, so it includes any wait before the row started.
- Each run's section writes its table first, right after the heading, with the run's `Idle` and span, so `TestCountOrch`'s pinned `"## run-a\n\n| role |"` still holds.
  After the table, with `Archived` set it writes a line giving the span's start, end and duration and the archive tag it was read from, then `writeProducers` of its `Producers`: `| producer | entries | elapsed | idle | share |`, share being elapsed over the run's span.
  With `Archived` nil it writes after the table the line `No shed status was read, so the span is the transcript window and idle covers only that window.` and no Producers table.
- The `writeTable` and `WriteMarkdown` doc comments name the new columns and rows.

Tests:
- `TestCountOrch`: run-b's and run-c's transcripts each gain a user line `[Request interrupted by user]` 30 seconds after their first line (11:00:30 and 13:00:30), which adds no message, so every charge and token figure stays.
  The `agent-b` fork, which names run-b and is charged wholly to it, moves to 11:29–11:30, inside run-b's window and still inside the orch span, so its charge stays.
  Run-b's and run-c's burler then each work 30s of a one-hour span, and the orch messages inside them, the fork charged wholly to run-b among them, fall in that run's idle.
  The pinned "All runs" rows become `| burler | 3 | 3000000 | 0 | 0 | 0 | 15.0M | 20.0% | 2h1m0s | 50.4% | $60.00 | claude-opus-5-5: 3 |\n| orch | | | | | | 60.0M | 80.0% | | | $240.00 | |\n| idle | | | | | | | | 1h59m0s | 49.6% | | |\n`, and run-a's become `| burler | 1 | 1000000 | 0 | 0 | 0 | 5.0M | 33.3% | 2h0m0s | 100.0% | $20.00 | claude-opus-5-5: 1 |\n| orch | | | | | | 10.0M | 66.7% | | | $40.00 | |\n| idle | | | | | | | | 0s | 0.0% | | |\n`.
  New pinned strings: the idle row `| idle | | | | | | | | 59m30s | 99.2% | | |` in both run-b's and run-c's sections, each run's no-status line, and no `| producer |` header anywhere, since no run has a status.
  `TestCountRun`'s `| webster+sub | … | 70.5% |` prefix and `TestEstimatedCost`'s `| $10.00 + unpriced | … |` suffix still match unchanged.
- `TestArchivedRunTiming` gains a render subtest: a `Report` of two runs, each with `Archived` and `Producers`, both holding a `Plan-Write` row, writes each run's span line naming its tag and its `writeProducers` rows with their elapsed, idle and share cells, the "All runs" Producers table whose `Plan-Write` row sums both runs' entries, elapsed and idle with its share over the two spans summed, and `timeRule`.

**ImpactSummary:** `Report.WriteMarkdown` is `writeTable`'s only caller (both calls updated); `TestCountOrch` is the only test pinning whole role-table rows, and the other report assertions match as substrings that the new columns leave intact.
