# Card 4 — Time columns, idle row and Producers tables in the report

**Create:**
- `plan:tools/tokencount#TableTime` -> `type TableTime struct`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, producers []ProducerTime, span time.Duration)`

**Edit:**
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, times TableTime)`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#TestCountOrch`
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:**
Decisions 4, 5, 7 and 10: render card 3's `RunTally.Time` in the report.

`TableTime` holds `Span, Idle time.Duration` and `Roles map[string]time.Duration` for one role table.
`writeTable` takes it as a new last parameter; its one caller, `Report.WriteMarkdown`, passes it for "All runs" and for each run.
The header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`; `share` keeps its weight meaning and rows keep their weight order.
A role row's `time` is `times.Roles[role]` rounded to the second and printed as a `time.Duration`, and `time share` is that time over `times.Span` as a percentage with one decimal, `0.0%` when the span is zero.
After the role rows and before the orch row comes one `idle` row: `| idle | | | | | | | | <idle> | <idle share> | | |`.
The orch row gets two blank time cells, so it reads `| orch | | | | | | <weight> | <share> | | | <cost> | |`.

`Report.WriteMarkdown`:
- "All runs" sums each run's `Time.Roles`, `Time.Idle` and span (`Time.To - Time.From`), each computed within its run, with no union across runs (Decision 7).
- Under the "All runs" table it states once: idle is the time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push; role time shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent; the union of working time plus idle is the span; and a producer's elapsed is the gap between finish times, so it includes any wait before the row started.
- Then a "Producers (all runs)" table, `writeProducers` over each producer name's entries, elapsed and idle summed across the runs whose `Time.HasStatus` is true, with share over the sum of those runs' spans; it is omitted when no run has a status.
- Each run's section prints its role table, then, when `Time.HasStatus`, its Producers table over its own span; when `Time.NoHistory`, the line `The archived shed status holds no history entries; the span is the transcript window and no Producers table is printed.`; and otherwise the line `No shed status was read; the span is the transcript window and no Producers table is printed.`
- `writeProducers` prints `| producer | entries | elapsed | idle | share |` with one row per producer in the given order, durations rounded to the second, share as elapsed over `span` with one decimal.

In `main.go`, `run`'s report title becomes `# Token usage and time by role`; the output path `.scratch/token-usage-by-role.md` stays.
The package doc in `main.go` gains the time measurement: working time is turn time (the three turn-ending line kinds), a role's time is the union of its transcripts' working intervals clipped to the run's span with `<role>+sub` its own row, the span's start and end and their bounds, idle, the Producers split, the orch row's blank time cells, and the new columns; one sentence per line (`PATTERN-comment-line-breaks`).
The description comments of `tokencount.sh` and `tokencount.cmd` say the report gives token usage and time by role, with the idle row and the Producers table.

`TestCountOrch` is the test that asserts whole role-table rows over several runs, so it carries the rendering rows:
- each run's `Time` is set with `MeasureRunTime` before the report is written: run-b's with `nil`; run-c's with an `ArchivedRun` literal whose status holds two history entries of two producers inside run-c's window; run-a's with an `ArchivedRun` literal whose status holds one history entry of a producer name run-c's history also holds, inside run-a's window;
- run-b's transcript gains an assistant line without usage at 11:10 with `stop_reason` `end_turn` and a user line at 11:50, so its burler works 20m of its 1h span; its table's idle row reads `40m0s` and `66.7%`;
- the orch sub-agent `agent-b`, whose prompt names run-b alone and which is charged wholly to run-b, moves from 13:29–13:30 to 11:20–11:25, inside run-b's 11:10–11:50 gap, with its comment saying it sits in run-a's and run-b's windows yet names run-b alone; the charge assertions stay as they are, since a sub-agent naming one run goes wholly to it at any time inside the orch window, and run-b's idle row still reads `40m0s`, which it would not if that sub-agent or orch's 11:30 message entered the run's union;
- the exact burler and orch rows it asserts gain the time cells, the orch row's blank;
- the "All runs" idle row and burler time are the sums over the three runs;
- run-b's section carries the no-status line, run-a's and run-c's carry their Producers tables, and "Producers (all runs)" holds the shared producer's row with the entries and elapsed of run-a and run-c summed, its share over the sum of run-a's and run-c's spans;
- the stated notes appear once.
`Busy` is filled only by `countFile`, which only `CountRun` calls, so orch transcripts cannot enter a run's union by construction; the run-b idle row pins that no later change routes them in.
`TestCountRun` and `TestEstimatedCost` assert row prefixes and the cost-and-models suffix, which the new middle columns leave intact.

**ImpactSummary:** `writeTable` has one caller, `Report.WriteMarkdown`; the report's table shape changes, which `TestCountOrch` re-pins, and the launchers change in comments only.

**Commit:** 4: tokencount: time columns, idle row and Producers tables
