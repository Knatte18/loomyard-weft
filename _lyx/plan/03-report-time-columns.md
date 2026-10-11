# Card 3 — The report shows time, idle and the Producers split

**Create:**
- `plan:tools/tokencount#tableTime` -> `type tableTime struct`
- `plan:tools/tokencount#formatDuration` -> `func formatDuration(d time.Duration) string`
- `plan:tools/tokencount#timeShare` -> `func timeShare(d, span time.Duration) string`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, producers []ProducerTime, span time.Duration)`
- `plan:tools/tokencount#timeRule` -> `const timeRule = ""`

**Edit:**
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, times tableTime)`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#TestCountOrch`
- `plan:tools/tokencount#TestRunTime`

**Intent:**
Decisions 4, 5 and 7: the report prints what card 2 measures.
All of it lives in `tools/tokencount/count.go` beside `writeTable`.

`tableTime` carries one role table's time: `Span`, `Idle` and `Roles map[string]time.Duration`.
`formatDuration(d)` is `d.Round(time.Second).String()`.
`timeShare(d, span)` is `100*d/span` printed `%.1f%%`, and `0.0%` when `span` is zero.

`writeTable` gains the `times` parameter.
Its header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`, and the separator line gains two `|---` cells.
`share` keeps its meaning, the weight share, and rows keep their weight order.
Each role row prints `formatDuration(times.Roles[role])` and `timeShare` of it over `times.Span` after `share`, so a role with no working time prints `0s` and `0.0%`.
The orch row prints its two time cells blank: `| orch | | | | | | <weight> | <share> | | | <cost> | |`.
After the orch row, or after the last role row when there is none, comes the `idle` row: `| idle | | | | | | | | <formatDuration(times.Idle)> | <timeShare(times.Idle, times.Span)> | | |`.
The `idle` row is last, so the role and orch rows above it are unchanged in order and adjacency.

`WriteMarkdown` builds each table's `tableTime`:
- Per run: `run.Time.Span` is `run.Time.To.Sub(run.Time.From)`, with `run.Time.Roles` and `run.Time.Idle`.
- "All runs": each role's time, the idle and the span are the sums over the runs of the per-run figures; no union is taken across runs (Decision 7).

Below the per-run table it writes, in order:
- When `!run.Time.FromStatus`: `No shed status was read; the span is the transcript window.`
- Always: `Span <formatDuration> from <From> to <To>.` with both instants in UTC `time.RFC3339`.
- When `run.Time.FromStatus`: a `### Producers` heading and `writeProducers(w, run.Time.Producers, span)`.

In "All runs", after the existing weight and unpriced lines, it writes `Total span <formatDuration> over <N> runs.` and then the `timeRule` paragraph.
When any run has `FromStatus`, it then writes a `### Producers` table summed over those runs by producer name (entries, elapsed and idle added), with shares over the "All runs" span.

`writeProducers` writes `| producer | entries | elapsed | idle | share |`, its separator, and one row per producer, sorted by elapsed descending and then by name: `| <name> | <entries> | <elapsed> | <idle> | <timeShare(elapsed, span)> |`.

`timeRule` is the report's one statement of how time is measured, in one paragraph.
- Time is turn time: a session works from a line until the next one, unless the line ends its turn.
- A role's time is the union of its sessions' working time, clipped to the run's span.
- Role time shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent's.
- The union of all working time plus idle is the span.
- Idle is time no session of the run is in a turn: parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push.
- A producer's elapsed is the gap between its history entry's finish time and the latest earlier finish time, so it includes any wait before the row started.

Tests:
- `TestCountOrch` asserts two exact role-table lines; both gain the time columns.
  `| burler | 3 | 3000000 | 0 | 0 | 0 | 15.0M | 20.0% | $60.00 | claude-opus-5-5: 3 |\n| orch | | | | | | 60.0M | 80.0% | $240.00 | |\n` becomes the same burler row with the run's two time cells inserted after `20.0%`, followed by `| orch | | | | | | 60.0M | 80.0% | | | $240.00 | |\n`, which pins the blank orch time cells.
  The `run-a` role-table line at `tools/tokencount/orch_test.go:135` changes the same way; the Orchestrator-section line `| run-a | 2.0M | $40.00 | $20.00 | 66.7% |` is not a role table and stays.
  Its runs come from `CountRun` with no `MeasureTime`, so their time cells are `0s` and `0.0%`.
  `TestCountRun`'s `| webster+sub | 3 | 10 | 0 | 18 | 4 | 0.0M | 70.5% |` and `TestEstimatedCost`'s `| $10.00 + unpriced | claude-future-9: 1, claude-sonnet-5-5: 1 |` still match, since time cells sit between `share` and `est. cost` and neither substring spans that point.
- `TestRunTime` gains a rendering block after its table.
  The table loop keeps each row's measured `RunTally` in a map keyed by row name, outside the subtests, which run serially.
  The block writes `Report{Runs: ...}` over two of them: the status-span row and the two-turns row, slugged apart.
  Both rows' transcripts carry a role of the same name, so "All runs" sums one role across them.
  It asserts these exact substrings:
  - the new header line;
  - each run's `idle` row: `| idle | | | | | | | | 1h40m0s | 83.3% | | |` for the status run and `| idle | | | | | | | | 1h50m0s | 73.3% | | |` for the two-turns run;
  - the status run's role row time cells;
  - `Span 2h0m0s from <From> to <To>.` for the status run;
  - `No shed status was read; the span is the transcript window.`, present in the two-turns run's section and absent from the status run's;
  - the status run's `### Producers` table, rows in the order B, A, D, C;
  - the "All runs" `idle` row: `| idle | | | | | | | | 3h30m0s | 77.8% | | |`;
  - the "All runs" row of the shared role, whose time cells are its 20m and 40m summed, `1h0m0s`, with share `22.2%` over 4h30m;
  - `Total span 4h30m0s over 2 runs.`;
  - `timeRule`;
  - the "All runs" Producers rows with shares over 4h30m.

**ImpactSummary:** `writeTable`'s two callers are both in `Report.WriteMarkdown`, edited here; the report's role tables gain two columns and an `idle` row, which changes the exact lines `TestCountOrch` asserts and no other test's.

**Commit:** 3: tokencount: time columns, idle row and Producers tables
