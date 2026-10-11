# Card 4 — Time columns, the idle row and the Producers tables in the report

**Create:**
- `plan:tools/tokencount#covered` -> `func covered(spans, cover []interval) time.Duration`
- `plan:tools/tokencount#tableTime` -> `type tableTime struct`
- `plan:tools/tokencount#RunTally.tableTime` -> `func (run RunTally) tableTime() tableTime`
- `plan:tools/tokencount#producerRow` -> `type producerRow struct`
- `plan:tools/tokencount#RunTally.producerRows` -> `func (run RunTally) producerRows() []producerRow`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, rows []producerRow, span time.Duration)`
- `plan:tools/tokencount#roundedDuration` -> `func roundedDuration(d time.Duration) string`
- `plan:tools/tokencount#timeRule` -> `const timeRule = ""`

**Edit:**
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, clock tableTime, orch *OrchCharge)`
- `tools/tokencount#TestCountRun`
- `tools/tokencount#TestCountOrch`

**Intent:**
Decisions 2 (the clip), 4, 5 (the table), 7 and the reporting half of 3: the report shows each role's time beside its tokens, the run's idle time, and the Producers split.

In `worktime.go`:
- `covered(spans, cover)` is the length of the part of `spans` that `unionOf(cover)` covers; `spans` are disjoint.
- `roundedDuration(d)` is `d.Round(time.Second).String()`.

In `count.go`:
- `tableTime` holds `Roles map[string]time.Duration`, `Idle` and `Span` (`time.Duration`).
- `RunTally.tableTime()` computes, within the run alone: each role's time `covered([]interval{run.Span}, role.work)`, the clip of Decision 2; the run-wide union `all` of every role's `work`, `+sub` rows included; `Idle` as the span's length minus `covered([]interval{run.Span}, all)` (Decision 4); `Span` as the span's length.
  The run's roles come only from its own worktree's transcripts (`CountRun`), so no orch session or orch sub-agent enters `all`, including one `CountOrch` charges wholly to the run.
- `producerRow` holds `Producer`, `Entries`, `Elapsed` and `Idle`; `RunTally.producerRows()` gives one per `ProducerTime` in order, `Elapsed` its `lengthOf(spans)` and `Idle` its elapsed minus `covered(spans, all)`.
- `timeRule` is the report's statement of how time is measured, printed once below the "All runs" table: time is turn time, from a user or assistant line to the line that ends the session's turn; a role's time is the union of its sessions' working time within the run's span; idle is the part of the span in which no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push; time shares divide by the span, so role shares can sum past 100% where sessions overlap (a `+sub` row overlaps its parent), and the union of working time plus idle is the span; a producer's elapsed is the gap between its history entry's finish time and the latest earlier one, so it includes any wait before the row started.
  The sentences above are the content; the constant is one Go string.
- `writeTable` takes the table's `tableTime`.
  Its header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`; `share` keeps its weight meaning and rows keep their weight order.
  Each role row's `time` is `roundedDuration(clock.Roles[role])` and `time share` is `%.1f%%` of it over `clock.Span` (0.0% when the span is zero).
  Below the role rows comes `| idle | | | | | | | | <idle> | <share> | | |`, then the orch row, whose `time` and `time share` cells are blank: `| orch | | | | | | <weight> | <share> | | | <cost> | |`.
- `writeProducers` writes `| producer | entries | elapsed | idle | share |`, one row per `producerRow`, `share` being elapsed over `span`.
- `Report.WriteMarkdown`:
  - "All runs" sums each run's `tableTime` (role times, idle, span; no union across runs) and passes the sum to `writeTable`, then prints `timeRule`.
    When any run has `StatusRead`, it then prints `### Producers` and `writeProducers` over the per-name sums of those runs' `producerRows` (entries, elapsed, idle; names in first-seen order), its share over the sum of those runs' spans.
  - Each run's section prints its table with the run's own `tableTime`, then a `Span <duration>, <start> to <end>.` line (times as RFC 3339 UTC), so the heading still sits right above the table as `TestCountOrch`'s `"## run-a\n\n| role |"` assertion expects, then `### Producers` and `writeProducers` with the run's span when `StatusRead`, else the line `<NoStatus>; the span is the transcripts' window, so idle covers only that window.`

Tests:
- `TestCountRun`'s `/hub/times` run from card 2 is rendered in a `Report` of two copies: `times` with `Span` 09:00–17:00, `StatusRead` true, `NoStatus` empty and `Producers` `Plan` (one entry, `[09:00, 12:36)`) and `Webster` (one entry, `[12:36, 17:00)`); and `times-clipped` with `Span` 10:30–16:00 and `NoStatus` `No shed status was read`.
  Card 2's block already pins the unclipped role unions, so these assertions pin what rendering adds, as exact rows or cells:
  - `times`: the `burler` row's `50m0s | 10.4%` cells; the idle row `6h39m0s | 83.1%` (480m span minus the 81m union); `Plan` `| 1 | 3h36m0s | 2h46m0s | 45.0% |` and `Webster` `| 1 | 4h24m0s | 3h53m0s | 55.0% |`; the span line.
  - `times-clipped`: `burler` 20m0s (clipped to 10:30–10:40 and 12:00–12:10), `reviewer` 10m0s (16:00–16:05 clipped away), the idle row `4h44m0s | 86.1%` (330m span minus the 46m clipped union), and the `No shed status was read; ...` line.
  - "All runs": `burler` `1h10m0s`; the idle row `11h23m0s | 84.3%` (399m + 284m over the 810m summed span); the Producers sum's `Plan` and `Webster` rows with the shares of `times` alone, 45.0% and 55.0%, since only `times` read a status; and `timeRule`.
  The existing `webster+sub` row assertion still matches as a prefix.
- `TestCountOrch`'s exact role-table rows gain the new cells: every run there comes from `CountRun` alone, which card 3 makes set the transcript-window span (run-a 2h, run-b 1h, run-c 1h), each burler session works its whole window, and the rows read `| burler | 3 | 3000000 | 0 | 0 | 0 | 15.0M | 20.0% | 4h0m0s | 100.0% | $60.00 | claude-opus-5-5: 3 |`, `| idle | | | | | | | | 0s | 0.0% | | |` and `| orch | | | | | | 60.0M | 80.0% | | | $240.00 | |` in "All runs", and run-a's likewise with 2h0m0s.
  It gains a block over a fresh projects directory: a run `run-idle` whose burler session works 10:00–10:10 and 11:00–11:10 (`end_turn` on each assistant line, through card 2's `turnLine`), and a prime holding an orch session and an orch sub-agent whose first prompt names `run-idle` alone, with orch messages at 10:30 and 10:40.
  After `CountOrch` charges the sub-agent wholly to `run-idle`, the rendered run section's idle row reads `50m0s` and `71.4%` of the 70m span, which fails if the orch messages' intervals entered the run's union.

**ImpactSummary:** `writeTable` has two callers, both in `Report.WriteMarkdown`; every role table gains two columns and an idle row, which changes the exact rows `TestCountOrch` asserts and no other test's.

**Commit:** 4: tokencount: time and time share per role, idle row, Producers tables
