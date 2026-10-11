# Card 2 — Turn time per role, idle, and the time columns

**Create:**
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#turnMark` -> `type turnMark struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(marks []turnMark) []interval`
- `plan:tools/tokencount#unionOf` -> `func unionOf(intervals []interval) []interval`
- `plan:tools/tokencount#coveredWithin` -> `func coveredWithin(union []interval, from, to time.Time) time.Duration`
- `plan:tools/tokencount#RunTally.SetSpan` -> `func (run *RunTally) SetSpan(start, end time.Time)`
- `plan:tools/tokencount#formatDuration` -> `func formatDuration(d time.Duration) string`
- `plan:tools/tokencount#TestWorkingTime` -> `func TestWorkingTime(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#CountRun`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, span, idle time.Duration)`
- `tools/tokencount#run`
- `tools/tokencount#TestCountOrch`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Measure each role's working time and the run's idle time from turn boundaries, and print them beside the tokens (Decisions 1, 2, 4, 7 and 10).
This card's span is the run's transcript window, `First`..`Last`; card 3 replaces it with the archived-status span through the same `SetSpan`.

Turn boundaries, in a new file `tools/tokencount/worktime.go` holding every new non-test symbol of this card:
- `line.Message` gains `StopReason string` with JSON key `stop_reason`; null and absent both decode as empty.
- `endsTurn(l)` is true for an assistant line whose `StopReason` is non-empty and not `tool_use`, and for a user line whose first text (`userTexts(l.Message.Content)[0]`) starts with `[Request interrupted by user` or with `<local-command-stdout>`.
  Every other line, a user line holding only tool results included, is false.
- `turnMark` holds a line's parsed timestamp and its `endsTurn` answer.
- `workingIntervals(marks)` stable-sorts the marks by timestamp and returns, for each consecutive pair, the interval from the first to the second unless the first ends the turn.
  A foreground tool call, an assistant `tool_use` line followed much later by its result, is therefore working time, and so is the gap before a crashed session's next line.
- `unionOf` merges overlapping and touching intervals into a sorted, disjoint list.
- `coveredWithin(union, from, to)` sums the parts of a union's intervals inside `[from, to]`, 0 when `to` is not after `from`.
- `formatDuration(d)` is `d.Round(time.Second).String()`.

Collection: `RunTally.countFile` collects a `turnMark` for every user or assistant line whose timestamp parses (the lines `observeTime` already parses, duplicates by message id included, since duplicates are a token concern only), then appends `workingIntervals` of that one transcript to an unexported per-role list `working map[string][]interval` on `RunTally`, keyed by the role it counts under (`<role>+sub` for a sub-agent, exactly as tokens), and to an unexported run-wide list.
Only `countFile` feeds them, so orch sessions and orch sub-agents, which `CountOrch` reads through `readTranscript`, never enter a union, an orch sub-agent charged wholly to the run included; the run's idle holds their time (Decision 4).

`RunTally` gains exported `SpanStart, SpanEnd time.Time` and `Idle time.Duration`; `RoleTally` gains `Time time.Duration`.
`SetSpan(start, end)` sets the span, stores the run-wide union as `unionOf` of the run-wide list (kept unexported for card 4's Producers idle), sets each role's `Time` to `coveredWithin(unionOf(working[role]), start, end)`, and sets `Idle` to the span minus `coveredWithin` of the run-wide union, so the union of working time plus idle is the span.
`CountRun` calls `run.SetSpan(run.First, run.Last)` after counting every transcript.

Rendering:
- `writeTable` takes the table's span and idle and adds `time` and `time share` columns after `share`: the header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`.
  A role's time share is `100 * Time / span` printed `%.1f%%`, `0.0%` when the span is 0.
  Below the role rows, in their unchanged weight order, comes `| idle | | | | | | | | <idle> | <share> | | |`, then the orch row with blank time cells: `| orch | | | | | | <weight> | <share> | | | <cost> | |`.
- `Report.WriteMarkdown` sums, for "All runs", each role's `Time`, every run's `Idle` and every run's span (`SpanEnd - SpanStart`), each computed within its run, with no union across runs, and passes the per-run span and idle to each run's table.
  Below the "All runs" table, after the total-weight line, it prints once: a role's time is the union of its sessions' turn time, so role time shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent; idle is time no session of the run is in a turn, which holds parent review, gates, gate-slot waits, a parked driver and Go-only work such as verify and push; and the union of working time plus idle is the span.
- `run` in `main.go` writes the title `# Token usage and time by role`; the output path is unchanged.

Package doc in `main.go`: the first sentence names time beside tokens, and a new paragraph after the share-column paragraph states Decision 1's turn rule (the three turn-ending line kinds, the interval after one idle until the next user or assistant line, a foreground tool call working), Decision 2's union per role and per `+sub` row clipped to the span, Decision 4's idle and the orch exclusion, Decision 7's columns, blank orch cells, rounding to the second and "All runs" sums, and that the span is the run's first to last transcript line (card 3 rewrites that sentence).

Tests:
- `TestWorkingTime`, new in `count_test.go`, table-driven over `CountRun` of synthetic transcripts whose lines all carry timestamps; closures inside the test, not package-level helpers, build a timed user text line and a timed assistant line with a given `stop_reason`, and the existing `toolResult` builds a timed tool result.
  Each row names its transcripts and wants per-role `Time`, `Idle` and the span:
  - two turns: user 10:00, assistant `end_turn` 10:05, user 12:00, assistant `end_turn` 12:10 gives the role 15m and idle 1h55m, the gap left out;
  - the interrupt marker: user 10:00, assistant `tool_use` 10:01, user `[Request interrupted by user for tool use]` 10:02, user 11:00, assistant `end_turn` 11:01 gives 3m, the gap after the marker idle;
  - a local slash command: assistant `end_turn` 10:00, user `<local-command-stdout>ok</local-command-stdout>` 10:01, user 11:00, assistant `end_turn` 11:02 gives 2m, nothing after the command output;
  - a foreground tool call: user 10:00, assistant `tool_use` 10:01, tool result 10:31, assistant `end_turn` 10:32 gives 32m;
  - overlap: two burler sessions working 10:00-10:30 and 10:20-10:50 give burler 50m, not 60m, and a sub-agent of a webster session working 10:05-10:15 counts 10m under `webster+sub` while `webster` keeps only its own transcript's time;
  - idle: in the overlap row, the run's span 10:00-10:50 and its working union 10:00-10:50 give idle 0s, and in the two-turn row idle plus the role's union equals the span.
  The test asserts `Time`, `Idle` and the span only; `TestCountOrch` pins the rendering.
- `TestCountOrch`: run-a's first fixture line becomes a `<local-command-stdout>` user line at 10:00, so run-a works no turn and its whole 2h window is idle, while the orch main session's 10:30 and 11:30 lines and the `agent-dup` sub-agent charged wholly to run-a fall inside that idle; charges are unchanged, since attribution reads only `First` and `Last`.
  It also asserts the "All runs" sentence's overlap clause, that role time shares can sum past 100% where sessions overlap.
  Its expected role-table strings gain the time cells and the idle row: "All runs" `| burler | 3 | 3000000 | 0 | 0 | 0 | 15.0M | 20.0% | 2h0m0s | 50.0% | $60.00 | claude-opus-5-5: 3 |\n| idle | | | | | | | | 2h0m0s | 50.0% | | |\n| orch | | | | | | 60.0M | 80.0% | | | $240.00 | |\n` and run-a `| burler | 1 | 1000000 | 0 | 0 | 0 | 5.0M | 33.3% | 0s | 0.0% | $20.00 | claude-opus-5-5: 1 |\n| idle | | | | | | | | 2h0m0s | 100.0% | | |\n| orch | | | | | | 10.0M | 66.7% | | | $40.00 | |\n`.
- `TestCountRun`'s `| webster+sub | 3 | 10 | 0 | 18 | 4 | 0.0M | 70.5% |` prefix and `TestEstimatedCost`'s row tail still match, so neither changes.
- `cmd/lyx/testdata/test-budget.yaml`: `tools/tokencount` rises from 12 to 13.

**ImpactSummary:** all changes are inside `tools/tokencount`, a `main` package no other package imports; `writeTable`'s two callers are both in `Report.WriteMarkdown`, and the only existing assertions on a full role-table row are `TestCountOrch`'s, updated here.

**Commit:** 2: tokencount: turn time per role, an idle row and the time columns
