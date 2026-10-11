# Card 6 — time columns, idle row and Producers table in the report

**Create:**
- `plan:tools/tokencount#RunTime.add` -> `func (t *RunTime) add(o RunTime)`
- `plan:tools/tokencount#formatDuration` -> `func formatDuration(d time.Duration) string`
- `plan:tools/tokencount#timeShare` -> `func timeShare(d, span time.Duration) float64`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, t RunTime)`
- `plan:tools/tokencount#TestReportTime` -> `func TestReportTime(t *testing.T)`

**Edit:**
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, times RunTime)`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#TestCountOrch`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#RunTime`
- `plan:tools/tokencount#ProducerTime`
- `plan:tools/tokencount#RunTally.Measure`
- `plan:tools/tokencount#assistantLine`
- `tools/tokencount#timedAssistant`
- `tools/tokencount#userPrompt`

**Intent:** The report prints what cards 3 to 5 measure: working time and time share per role, an `idle` row, and a Producers table.
`formatDuration` is `d.Round(time.Second).String()`; `timeShare` is `100 * d / span` as a percentage and 0 when the span is 0.
`RunTime.add` sums another run's `RunTime` into the receiver for "All runs": `Span`, `Idle`, `ProducerSpan` and each role's time add, and `Producers` merge by name with `Entries`, `Elapsed` and `Idle` summed; `Window` stays zero.
No union is taken across runs.

`writeTable` gains the `times RunTime` parameter, and `WriteMarkdown` is its only caller (the test files never call it), passing `run.Time` for a run and the sum for "All runs".
The table gains `time` and `time share` columns after `share`, so the header and separator have twelve columns; `share` keeps meaning the weight share, and rows keep their weight order.
A role row prints `formatDuration(times.Roles[role])` and `timeShare` of it over `times.Span`; the orch row leaves both time cells blank, since orch spans many runs.
An `idle` row ends every table, after the orch row, with blank cells from sessions through share, the idle time and its time share, and blank cost and models cells: `| idle | | | | | | | | 0s | 0.0% | | |`.

`WriteMarkdown` prints, in "All runs" below the table's totals line, one paragraph stating how the time columns are measured, one sentence per line in the Go string:
role time is the union of a role's sessions' turn time inside the run's span, a turn running from a user or assistant line to the next line unless that line's turn ended (an assistant `stop_reason` other than `tool_use`, an interrupt marker, or a local command's output);
a `+sub` row is the union of that role's sub-agent transcripts;
role shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent;
the union of working time plus idle is the span;
idle is time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push;
a run's span runs from its start to the latest end among its shed history entries, or is the transcript window when no shed status was read.
Each run's section gains, after its table, a line giving its span, with its start and end in UTC, and, when `StatusRead` is false, the sentence "No shed status was read, so the span is the transcript window, idle covers only that window and there is no Producers table."
When `Producers` is non-empty, "All runs" and the run's section each gain a `### Producers` heading and a table with columns `producer`, `entries`, `elapsed`, `idle` and `share`, the share being `Elapsed` over `ProducerSpan`, and the sentence that a row's elapsed is the gap between finish times, so it includes any wait before the row started.
The "All runs" Producers table sums only the runs whose status was read, and its paragraph says so.
The measurement paragraph, the elapsed sentence and the no-status sentence are Go constants, so the test can assert their key phrases without copying whole paragraphs.
The line giving a run's span sits after its table, so `"## run-a\n\n| role |"` in `TestCountOrch` still holds.
The doc comments of `Report.WriteMarkdown` and `writeTable` are updated for the time columns.

What this can mislead: idle also holds Go-only work, since the data cannot tell it from waiting; the printed definition says so, and that statement is the bound.

`TestCountOrch` asserts rows whose shape changes: its expected role-table rows gain the time cells and an `idle` row.
Its runs have one user line at the window start and one assistant line at the window end, with no `stop_reason`, so each run's working time is its whole window and idle is 0s: the "All runs" burler row reads `| burler | 3 | 3000000 | 0 | 0 | 0 | 15.0M | 20.0% | 4h0m0s | 100.0% | $60.00 | claude-opus-5-5: 3 |`, followed by the orch row `| orch | | | | | | 60.0M | 80.0% | | | $240.00 | |` and `| idle | | | | | | | | 0s | 0.0% | | |`, and run-a's burler row reads `... | 5.0M | 33.3% | 2h0m0s | 100.0% | $20.00 | claude-opus-5-5: 1 |` with its orch row `| orch | | | | | | 10.0M | 66.7% | | | $40.00 | |`.
The orch row's blank time cells are asserted there.
`TestCountRun`'s `"| webster+sub | 3 | 10 | 0 | 18 | 4 | 0.0M | 70.5% |"` stays a prefix of the new row, and `TestEstimatedCost`'s `"| $10.00 + unpriced | claude-future-9: 1, ..."` stays a substring, since est. cost and models stay adjacent; neither is edited.

`TestReportTime` is one new top-level test in `count_test.go` over a full report, where no existing test reaches a status, a Producers table or an orch session working during a run's idle.
Run `alpha` has a burler session (user 10:00, `end_turn` 10:30, user 11:30, `end_turn` 12:00, so 1h) with a sub-agent transcript working 10:10 to 10:20, and `Measure(&RunArchive{...})` with a seed `started_at` of 09:30 and history entries `Plan-Write` at 10:30 and `Plan-Review` at 12:30, so the span is 3h, busy time is 1h and idle is 2h0m0s.
An orch session in a prime directory works 10:35 to 11:25, with an orch sub-agent whose first prompt names `alpha` working 10:40 to 11:20, and `CountOrch` charges them to `alpha`.
Run `beta` has no archive: a burler session working 14:00 to 14:20 and 14:40 to 15:00, so its span is the 1h window and its idle 20m.
The test asserts the `alpha` table's time cells (burler 1h0m0s at 33.3%, burler+sub 10m0s at 5.6%, idle 2h0m0s at 66.7%), which fails if orch time entered the union, the blank orch time cells being `TestCountOrch`'s assertion alone; the `Producers` rows `Plan-Review` (2h0m0s elapsed, 1h30m0s idle, 66.7%) before `Plan-Write` (1h0m0s, 30m0s, 33.3%); `alpha`'s span line (09:30 to 12:30 UTC, 3h0m0s); the measurement paragraph's key phrases ("can sum past 100%", "no session of the run is in a turn", "plus idle is the span") and the elapsed sentence's ("gap between finish times"); `beta`'s no-status sentence and the absence of a `### Producers` heading in its section; and the "All runs" row sums (burler 1h40m0s at 41.7%, idle 2h20m0s at 58.3%, span 4h0m0s) and Producers shares over the 3h of `alpha`.
The `tools/tokencount` row in `test-budget.yaml` goes from 16 to 17.

**ImpactSummary:** `writeTable` is re-signed with one caller, `WriteMarkdown`, on this card; the only assertions on the old column layout are in `TestCountOrch`, edited here, and the report's table text changes for every run.
