# Card 2 — turn-time-columns

**Create:**
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#timedLine` -> `type timedLine struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(lines []timedLine) []interval`
- `plan:tools/tokencount#unionOf` -> `func unionOf(sets ...[]interval) []interval`
- `plan:tools/tokencount#clip` -> `func clip(ivs []interval, from, to time.Time) []interval`
- `plan:tools/tokencount#durationOf` -> `func durationOf(ivs []interval) time.Duration`
- `plan:tools/tokencount#uncovered` -> `func uncovered(from, to time.Time, covered []interval) time.Duration`
- `plan:tools/tokencount#RunSpan` -> `type RunSpan struct`
- `plan:tools/tokencount#RunTally.span` -> `func (run RunTally) span() RunSpan`
- `plan:tools/tokencount#RunTally.working` -> `func (run RunTally) working() []interval`
- `plan:tools/tokencount#RunTimes` -> `type RunTimes struct`
- `plan:tools/tokencount#RunTally.Times` -> `func (run RunTally) Times() RunTimes`
- `plan:tools/tokencount#timeCells` -> `func timeCells(d, span time.Duration) string`
- `plan:tools/tokencount#timeRule` -> `const timeRule = ""`
- `tools/tokencount/runtime_test.go`
- `plan:tools/tokencount#TestRunTime` -> `func TestRunTime(t *testing.T)`
- `plan:tools/tokencount#stoppedAssistant` -> `func stoppedAssistant(id, ts, stop string) string`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, times RunTimes)`
- `tools/tokencount#TestCountOrch`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Measure each role's working time from its transcripts and print it, with an `idle` row, in every role table (Decisions 1, 2, 4 and 7).
The span is the run's transcript window in this card; card 3 replaces it with the archived shed status's span where one is read.

A new file `tools/tokencount/runtime.go` holds the interval arithmetic and the time types.

Turn time (Decision 1):
- `line`'s anonymous `Message` struct gains `StopReason string` tagged `json:"stop_reason"`; a JSON null decodes to empty.
- `endsTurn` reports whether a line ends its session's turn: an `assistant` line whose `StopReason` is non-empty and not `tool_use`, or a `user` line one of whose `userTexts` starts with `[Request interrupted by user` or with `<local-command-stdout>`.
  An assistant line with an empty `stop_reason` does not end the turn, so a session that crashes mid-turn counts its gap as working, per the record's auto-mode assumption.
- `RunTally.countFile` collects a `timedLine` (timestamp and `endsTurn`) for every `user` and `assistant` line whose `timestamp` parses as RFC 3339, before its usage checks, so a duplicate message's line still counts as a line.
  After the walk it passes them to `workingIntervals`, which sorts them by timestamp (stable) and returns, for each pair of consecutive lines, the interval between them unless the first ends the turn.
  A foreground tool call is therefore working time: its `tool_use` assistant line does not end the turn, so the wait for its result counts.
- `RoleTally` gains an unexported `working []interval` field, and `countFile` folds each transcript's intervals into its role's tally through `unionOf`, so concurrent sessions of one role count once (Decision 2).
  Sub-agent transcripts already count under `<role>+sub` through `CountRun`, so `+sub` is its own union.

Span and idle:
- `RunSpan` holds `Start` and `End`; `RunTally` gains an exported `Span RunSpan` field.
- `RunTally.span` returns `Span` when `Span.End` is non-zero, else the transcript window `First`..`Last`.
- `RunTally.Times` clips each role's union to the span with `clip` and sums it with `durationOf` into `RunTimes.Roles`, keyed by role.
  `RunTally.working` is the union of every role's clipped intervals, and `RunTimes.Idle` is `uncovered(span.Start, span.End, run.working())`, the span minus that union (Decision 4).
  `RunTimes.Span` is the span's duration.
  Orch transcripts never enter a `RunTally`, since `CountOrch` reads the prime's project directory into an `OrchTally`, so an orch sub-agent charged wholly to a run adds nothing to its union and its time stays in the run's idle.

Table (Decision 7):
- `writeTable` gains a `times RunTimes` parameter, and its header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`, the two time columns right after `share`.
  Rows keep their weight order.
- `timeCells` renders `| <d> | <x.x%> |` for a duration over the table's span, the duration as `d.Round(time.Second)` and the share `0.0%` when the span is zero.
- A role row takes `timeCells(times.Roles[role], times.Span)`.
  The orch row's two time cells are blank.
  An `idle` row follows the orch row, or the role rows when there is none: `| idle | | | | | | | | <idle> | <idle share> | | |`.
  Below the table, before the `Total weight` line of "All runs", a line `Span <d>.` gives the table's span.
- `Report.WriteMarkdown` takes each run's `Times()` once, passes it to that run's `writeTable`, and for "All runs" sums every run's role times, idles and spans, each computed within its run, with no union across runs.
  After the "All runs" table it prints `timeRule`, the report's statement that a role's time is the union of its sessions' turns clipped to the run's span, `+sub` rows counted apart, so role time shares can sum past 100% where sessions overlap, that idle is time no session of the run is in a turn (parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push), and that the union of working time plus idle is the span.

Tests:
- New top-level `TestRunTime` in the new file `runtime_test.go`, table-driven over one project directory per row, each row's sessions written with `writeLines`, `userPrompt`, `toolResult` and the new helper `stoppedAssistant`, an assistant line at `ts` carrying `stop_reason` `stop` (empty writes a JSON null).
  Every session opens with a `userPrompt` line at the row's first time.
  Each row asserts the exact `Times()` roles map, idle and span:
  - prompts at 10:00 and 11:05, each answered by `end_turn` five minutes later: burler 10m, span 1h10m, idle 1h;
  - a prompt at 10:00, a `tool_use` at 10:01 answered by an interrupt-marker user line at 10:02, then a prompt at 11:00 and `end_turn` at 11:01: 3m, the 58 minutes after the marker idle;
  - a prompt at 10:00, `end_turn` at 10:01, a `<local-command-stdout>` user line at 10:30, a prompt at 11:00, `end_turn` at 11:01: 2m, the half hour after the local command idle;
  - a prompt at 10:00, a `tool_use` at 10:01 whose result lands at 10:31, then `end_turn` at 10:32: 32m, the foreground wait counted, idle 0;
  - two burler sessions working 10:00–10:20 and 10:10–10:30: burler 30m, not their 40m sum;
  - a session working 10:00–10:10 with a sub-agent working 10:02–10:04: burler 10m and burler+sub 2m, idle 0.
- `TestRunTime` then renders a `Report` of two of those runs with an orch tally from `CountOrch` over a prime directory holding an orch session and a sub-agent whose `userPrompt` names the first run's slug and whose lines sit in that run's idle hour.
  It asserts the first run's idle row is unchanged at `1h0m0s`, its orch row carries blank time cells, "All runs" sums the two runs' role times, idles and spans, and the `timeRule` text and the `Span` lines appear.
- `TestCountOrch` keeps its fixtures and changes only what the new columns force: its two exact role-table row assertions gain the two time cells after the share cell, blank on the orch rows.
  Its runs end on assistant lines with no `stop_reason`, so each burler row's time is its run's whole window at `100.0%`.
  Idle and orch exclusion stay with `TestRunTime`, since `TestCountOrch`'s orch sub-agent naming run-b works at 13:30, outside run-b's window.
- `tools/tokencount`'s row in `cmd/lyx/testdata/test-budget.yaml` rises from 12 to 13.

**ImpactSummary:** Adds two columns and an idle row to every role table; `writeTable`'s one caller is `Report.WriteMarkdown`, and `TestCountOrch` is the one test asserting whole role rows (`TestCountRun` and `TestEstimatedCost` match row prefixes and suffixes the new columns leave intact).
