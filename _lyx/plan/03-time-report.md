# Card 3 — time-report

**Create:**
- `plan:tools/tokencount#timeRule` -> `const timeRule = ""`
- `plan:tools/tokencount#producerRule` -> `const producerRule = ""`
- `plan:tools/tokencount#durationText` -> `func durationText(d time.Duration) string`
- `plan:tools/tokencount#timeShare` -> `func timeShare(d, span time.Duration) string`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, rows []ProducerTime, span time.Duration)`

**Edit:**
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, span, idle time.Duration)`
- `tools/tokencount#Report.WriteMarkdown`
- `plan:tools/tokencount#TestMeasureTime`
- `tools/tokencount#TestCountOrch`

**Uses:**
- `plan:tools/tokencount#RunTally.MeasureTime`
- `tools/tokencount#CountOrch`
- `plan:tools/tokencount#ProducerTime`

**Intent:**
Render the time card 2 measures: `time` and `time share` columns and an `idle` row in every role table, a Producers table per run and in "All runs", and the rules that say how each is measured (Decisions 4, 5 and 7).
`timeRule`, `producerRule`, `durationText`, `timeShare` and `writeProducers` go in `timing.go` beside the measurement.

`durationText` prints `d.Round(time.Second).String()`.
`timeShare` prints `100*d/span` as `%.1f%%`, and `n/a` when `span` is 0.

`writeTable` gains the table's `span` and `idle`.
Its header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`, with the separator row widened to match.
Each role row prints its `Time` through `durationText` and its share of `span` through `timeShare` after the `share` cell; rows keep their weight order and `share` keeps its weight meaning.
The orch row prints both time cells blank.
After the orch row (after the role rows when there is none) comes `| idle | | | | | | | | <idle> | <idle share> | | |`.
Update `writeTable`'s doc comment to say what the time columns divide by.

`Report.WriteMarkdown`:
- "All runs" sums each run's role `Time` into the summed role tallies, and sums the runs' spans (`SpanTo - SpanFrom`) and `Idle`, each computed within its run, with no union across runs; it passes those sums to `writeTable`.
- After the "Total weight" line it prints `timeRule`.
  `timeRule` states: a role's time is the time its sessions spend in a turn, from a prompt or a tool result to the line that ends the turn, a foreground tool call included; concurrent sessions of a role count once; idle is the time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push; time shares divide by the run's span, from the seed's start (or the run's earliest transcript line) to the last shed history entry's finish, and in "All runs" by the sum of the runs' spans; role shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent, while the union of all sessions' working time plus idle is the span; the orch row has no time, since orch spans many runs.
- When at least one run has a `ShedTag`, "All runs" then prints `### Producers` and `writeProducers` over the runs with one: one row per producer name, in order of first appearance across the runs as listed, its `Entries`, `Elapsed` and `Idle` summed, its share over the sum of those runs' spans; then `producerRule`.
  `producerRule` states that a producer row's elapsed is the gap between the finish times of consecutive shed history entries, so it includes any wait before the row started, that its idle is the part of that gap no session of the run works in, and that elapsed sums to the span.
- Each run's section, after its role table (passed the run's own span and `Idle`), prints the span: `Span <from> to <to> (<duration>), from the shed status at <tag>.` with `### Producers` and `writeProducers` over the run's `Producers` and span below it, or, when `ShedTag` is empty, `No shed status was read, so the span is the transcript window, <from> to <to> (<duration>), and there is no Producers table.`
  Times print as the Orchestrator section prints its own, `.Local().Format("2006-01-02 15:04")`.
- Update its doc comment to name the time columns, the idle row and the Producers tables.

`writeProducers` prints `| producer | entries | elapsed | idle | share |`, its separator row, one row per entry of `rows`, and a blank line.

Tests:
- `TestMeasureTime` rows gain a list of exact report lines each asserts after rendering `Report{Runs: []RunTally{run}}`: the idle row of the first row (`| idle | | | | | | | | 1h55m0s | 93.5% | | |`), the `### Producers` rows of the record row (`| A | 2 | 5m0s | 5m0s | 10.0% |`, `| B | 1 | 25m0s | 15m0s | 50.0% |`, `| C | 1 | 20m0s | 20m0s | 40.0% |`) and the no-status sentence of a row without a record.
- `TestMeasureTime` gains one row with an orch: the prime's project directory holds an orch session and an orch sub-agent whose first prompt names only the row's slug, both working inside the run's idle gap; the row runs `CountOrch` over the measured run and renders `Report{Runs, Orch}`, and asserts the run's idle row still holds the whole gap and the orch row's time cells are blank (`| orch | | | | | | <weight> | <share> | | | <cost> | |`).
- `TestMeasureTime` ends by rendering one `Report` over two of its measured runs and asserting the "All runs" table: a role present in both sums its two times, the idle row sums the two idles, and each time share divides by the two spans' sum.
- `TestCountOrch`'s exact role-table strings gain the two time cells: its runs are not measured, so each burler row reads `| 0s | n/a |` after its share, the orch row two blank cells, and the idle row follows the orch row.

**ImpactSummary:** `writeTable`'s one caller is `Report.WriteMarkdown`, re-wired here; the role tables gain two columns and a row, and `TestCountOrch` is the only test whose exact row strings cross the new cells (`TestCountRun`'s webster+sub assertion ends at the share cell and stays true).
