# Card 5 — tokencount-time-report

**Create:**
- `plan:tools/tokencount#tableTimes` -> `type tableTimes struct`
- `plan:tools/tokencount#reportTitle` -> `const reportTitle = ""`
- `plan:tools/tokencount#timeMeasureRule` -> `const timeMeasureRule = ""`
- `plan:tools/tokencount#producerRule` -> `const producerRule = ""`
- `plan:tools/tokencount#noShedStatusRule` -> `const noShedStatusRule = ""`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, producers []ProducerTime, span time.Duration)`
- `plan:tools/tokencount#TestReportTimes` -> `func TestReportTimes(t *testing.T)`

**Edit:**
- `tools/tokencount#Report`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, times tableTimes)`
- `tools/tokencount#run`
- `tools/tokencount#TestCountOrch`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** The report prints the time columns, the `idle` row and the Producers tables (Decisions 7 and 10).
`Report` gains `Generated time.Time`, and `WriteMarkdown` now opens with the title `# Token usage and time by role` (the constant `reportTitle`, which keeps the path `.scratch/token-usage-by-role.md` unchanged) and the `Generated <time> for <slugs>.` line that `run` printed; `run` drops that `fmt.Fprintf`, sets `Generated: time.Now()` on the `Report` it builds, and loses its now-unused `strings` import.
`tableTimes` holds `Roles map[string]time.Duration`, `Idle` and `Span`, and `writeTable` takes it as a fourth parameter: the header gains `time` and `time share` after `share`, and every role row prints its `Roles` duration rounded to the second beside its share of `Span` (0 when `Span` is 0).
The orch row's two time cells stay blank, and an `idle` row follows it with blank token cells and the idle time and share; the only caller of `writeTable` is `WriteMarkdown`, which passes each run's `RunTally.Times()` and, for "All runs", the sums over runs of each role's time, of `Idle` and of `Span`, with no union taken across runs.
Under each run's heading `WriteMarkdown` prints `Span <duration>, <start> to <end>.` with UTC times, then the table; a run with `NoStatus` then prints `noShedStatusRule`, which says no shed status was read, that its span is the transcript window and that idle covers only that window, and otherwise a `### Producers` table.
`writeProducers` prints `| producer | entries | elapsed | idle | share |` with the share being elapsed over the given span.
"All runs" prints, after its table and weight line, `timeMeasureRule` (idle is the time no session of the run is in a turn, so it holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work; role shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent; the union of working time plus idle is the span) and, when any run has producers, `producerRule` (a row's elapsed is the gap between finish times, so it includes any wait before the row started; the shares divide by the summed spans of the runs with a shed status) and a `### Producers` table summing entries, elapsed and idle by producer name over those runs.
The four rule constants are declared with `const` holding the sentences, replacing the empty literals above, and the test asserts the output contains each constant, as `orch_test.go` does for `orchAttributionRule`.
The doc comment of `WriteMarkdown` is rewritten to name the title, the span lines, the time columns and the Producers tables.
This adds no flag or guard; a run whose status was not read prints no Producers table rather than an empty one.
`TestCountOrch` is edited because its heading assertion `## run-a\n\n| role |` now reads `## run-a\n\nSpan 2h0m0s, 2026-01-02 10:00 UTC to 2026-01-02 12:00 UTC.\n\n| role |`, and because its role rows now carry the time cells: its run windows hold one user line and one assistant line with no turn end between them, so each run is working for its whole window, the "All runs" row reads `| 15.0M | 20.0% | 4h0m0s | 100.0% | $60.00 |` with the orch row `| 60.0M | 80.0% | | | $240.00 | |` and an `| idle | | | | | | | | 0s | 0.0% | | |` row after it, and run-a's rows read `| 5.0M | 33.3% | 2h0m0s | 100.0% | $20.00 |` and `| 10.0M | 66.7% | | | $40.00 | |`.
`TestReportTimes` renders `Report` values built from `RunTally` literals and asserts exact rows:
- run `alpha` with a `Shed` (start 10:00, entries `Plan-Write` 10:20, `Plan-Review` 10:40, `Publish` 11:00), `burler` working 10:05 to 10:25 and `burler+sub` 10:10 to 10:15:00.6 (so the rounding to the second is pinned), each with one million output tokens; it asserts the title, the span line, the `burler` row with `20m0s | 33.3%`, the `burler+sub` row with `5m1s | 8.4%`, the idle row `40m0s | 66.7%` and Producers rows `Plan-Review | 1 | 20m0s | 15m0s | 33.3%`, `Plan-Write | 1 | 20m0s | 5m0s | 33.3%` and `Publish | 1 | 20m0s | 20m0s | 33.3%` in that order;
- run `beta` with no `Shed`, a window of 30 minutes and 10 minutes of work; it asserts `noShedStatusRule` and that exactly two `### Producers` headings print, alpha's and "All runs";
- "All runs" over both: `burler` at `30m0s | 33.3%`, the idle row at `1h0m0s | 66.7%`, and the same three producers with shares over alpha's span alone;
- a run `gamma` counted from real transcripts through `CountRun` (a user line at 10:00, an assistant `end_turn` at 10:10, a user line at 11:55, an assistant `end_turn` at 12:00, built with `turnLine`) with a `Shed` ending at 12:00, and a prime directory holding an orch main-session message at 10:30 and an orch sub-agent message at 11:00 whose first prompt names `gamma`, written with `timedAssistant` and `userPrompt` as `TestCountOrch` does, counted through `CountOrch`; it asserts the orch row's time cells are blank and the idle row reads `1h45m0s | 87.5%`, so orch work stays in idle.
`cmd/lyx/testdata/test-budget.yaml` raises the `tools/tokencount` row by one for `TestReportTimes`.

**ImpactSummary:** `writeTable` has one caller, `WriteMarkdown`, in this file; the rows `TestCountOrch` asserts are the only tests that match the table's full width, since `TestCountRun`'s `| webster+sub | 3 | 10 | 0 | 18 | 4 | 0.0M | 70.5% |` and `TestEstimatedCost`'s `| $10.00 + unpriced | ...` fragments still match, and `run` is the only other writer of the report's first lines.
