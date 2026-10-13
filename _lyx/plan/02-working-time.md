# Card 2 — working-time

**Create:**
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#covered` -> `func covered(ivs []interval, span interval) time.Duration`
- `plan:tools/tokencount#turnLine` -> `type turnLine struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(lines []turnLine) []interval`
- `plan:tools/tokencount#spanOf` -> `func spanOf(run RunTally) (interval, string)`
- `plan:tools/tokencount#runTiming` -> `type runTiming struct`
- `plan:tools/tokencount#timingOf` -> `func timingOf(run RunTally) runTiming`
- `plan:tools/tokencount#tableTiming` -> `type tableTiming struct`
- `plan:tools/tokencount#formatDuration` -> `func formatDuration(d time.Duration) string`
- `plan:tools/tokencount#timeRule` -> `const timeRule = ""`
- `plan:tools/tokencount#TestRunTime` -> `func TestRunTime(t *testing.T)`
- `plan:tools/tokencount#userAt` -> `func userAt(t *testing.T, ts, text string) string`
- `plan:tools/tokencount#assistantAt` -> `func assistantAt(ts, stopReason string) string`
- `plan:tools/tokencount#timeCells` -> `func timeCells(t *testing.T, report, section, role string) (string, string)`
- `tools/tokencount/doc.go`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, timing tableTiming)`
- `tools/tokencount#TestCountOrch`
- `tools/tokencount/main.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#userTexts`
- `tools/tokencount#eachLine`
- `tools/tokencount#OrchCharge`

**Intent:**
Measure each role's working time and each run's idle time from the transcripts, and print them as new columns and an `idle` row of the role tables (Decisions 1, 2, 4 and 7).
This card's span is always the run's transcript window, `RunTally.First` to `RunTally.Last`; card 4 replaces it with the span read from the archived shed status.

Turn detection (Decision 1):
- `line`'s `Message` struct gains `StopReason string` with JSON key `stop_reason`; a null decodes as empty.
- `endsTurn` lives in a new file `tools/tokencount/timing.go`, beside every other new production member of this card.
  It is true for an assistant line whose `StopReason` is non-empty and not `tool_use`, and for a user line any of whose `userTexts` starts with `[Request interrupted by user` or with `<local-command-stdout>`.
- `turnLine` is one user or assistant line's parsed timestamp and whether `endsTurn` holds for it.
- `workingIntervals` stable-sorts the lines by timestamp, and each line that does not end a turn opens a working interval to the next line, empty intervals dropped; the last line opens none.
  So a foreground tool call, an assistant line with stop reason `tool_use` up to its tool result, is working time, and the gap after a turn-ending line is not.
- `RunTally.countFile` collects a `turnLine` for every user or assistant line whose timestamp parses (the same lines `observeTime` reads), runs `workingIntervals` over them once the file is read, and appends the result to the role's tally and to the run.
  `RoleTally` gains an unexported `working []interval` field, the working intervals of every transcript counted under that role, and `RunTally` gains an unexported `working []interval` field, those of every transcript of the run.
  A sub-agent transcript lands under `<role>+sub` as its tokens do; orch transcripts are never counted by `CountRun`, so they reach neither field (Decision 4).

Time figures:
- `interval` is a `from`, `to` pair of `time.Time`.
  `covered` is the length of the union of `ivs` clipped to `span`: merge the clipped intervals sorted by start, so overlapping sessions of one role count once (Decision 2).
- `spanOf` returns the run's span and the phrase saying where it came from; in this card it is `interval{run.First, run.Last}` and "the transcript window, first to last line".
- `runTiming` holds a run's span, that phrase, each role's time (`covered` of the role's intervals over the span) and its idle (the span's length less `covered` of the run's intervals); `timingOf` computes it.
- `tableTiming` holds what one role table prints: the span's length, the time per role name and the idle time.
- `formatDuration` is `d.Round(time.Second).String()`.

Report (Decision 7):
- `writeTable` takes the table's `tableTiming`.
  Its header gains `time` and `time share` after `share`: `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`.
  Each role row prints the role's `formatDuration` time and `100 × time / span` as `%.1f%%`, 0 when the span is 0; rows keep their weight order.
  The orch row's two time cells are blank: `| orch | | | | | | <weight> | <share> | | | <cost> | |`.
  A last row `| idle | | | | | | | | <time> | <share> | | |` prints the idle time and its share.
- `Report.WriteMarkdown` computes `timingOf` once per run.
  "All runs" passes the sum of the runs' spans, of each role's per-run times and of the per-run idles; no union is taken across runs.
  Below the "All runs" table and its "Total weight" line it prints `Total span <d> over <n> runs.` and then `timeRule` as its own paragraph.
  Each run's table is followed by `Span <d>, <phrase>.`
- `timeRule` states: time is wall-clock time a role's sessions spend in a turn, a turn ending at an assistant line whose stop reason is set and is not `tool_use`, at a user interrupt or at a local command's output; a role's time is the union of its sessions' turn time within the run's span; time share divides by the span, so shares sum past 100% where sessions overlap, as a `+sub` row overlaps its parent; idle is the span less the union of every session's turn time, the time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push; the union of turn time plus idle is the span; orch spans many runs, so its row has no time and its work counts as idle in every run.

Docs:
- The package doc moves out of `main.go` into a new `tools/tokencount/doc.go`, so later cards can name `main.go`'s `run` beside a doc change without listing the whole file.
  Moving it rewrites every line, so it is reflowed to one sentence per line (`PATTERN-comment-line-breaks`), wording unchanged but for the additions below.
- The doc gains a paragraph on the time columns and the idle row: how a turn ends, the union per role and per run, the span (for now, the transcript window), the time share's denominator, the "All runs" sums, and the orch row's blank cells.

Tests:
- `TestRunTime`, a new top-level test in `count_test.go`, writes one project directory per scenario under `t.TempDir()`, counts each with `CountRun`, writes one `Report{Runs: ...}` and reads the time cells with `timeCells`.
  `userAt` builds a user line at `ts` whose content is the string `text`; `assistantAt` builds an assistant line at `ts` with the given `stop_reason` and no usage, so no token column moves; `timeCells` returns the `time` and `time share` cells of the row whose first cell is `role` in the first table after the heading `## <section>`, failing the test when there is none.
  Scenarios, each with a `ly:<slug>:burler` title:
  - `gap`: user 10:00, `end_turn` 10:10, user 11:00, `end_turn` 11:20: burler `30m0s` `37.5%`, idle `50m0s` `62.5%`, and the section holds `Span 1h20m0s`.
  - `interrupt`: user 10:00, user `[Request interrupted by user]` 10:05, user 10:45, `end_turn` 10:50: burler `10m0s` `20.0%`, idle `40m0s` `80.0%`.
  - `local`: user 10:00, `end_turn` 10:10, user `<local-command-stdout>Set model</local-command-stdout>` 10:20, user 10:40, `end_turn` 10:50: burler `20m0s` `40.0%`, idle `30m0s` `60.0%`.
  - `tool`: user 10:00, `tool_use` 10:01, a tool result 10:30 (the existing `toolResult` helper), `end_turn` 10:31: burler `31m0s` `100.0%`, idle `0s` `0.0%`.
  - `overlap`: `burler` session user 10:00, `end_turn` 10:30; `burler-2` session user 10:15, `end_turn` 10:45; a sub-agent of the first, user 10:50, `end_turn` 11:00: burler `45m0s` `75.0%` (the union, not the 60-minute sum), `burler+sub` `10m0s` `16.7%`, idle `5m0s` `8.3%`.
  - "All runs": burler `2h16m0s` `50.2%`, `burler+sub` `10m0s` `3.7%`, idle `2h5m0s` `46.1%`, and the report holds `Total span 4h31m0s over 5 runs.` and `timeRule`.
- `TestCountOrch` keeps its runs and orch fixtures, and adds two user lines to `run-b`'s transcript, an interrupt marker at 11:10 and a prompt at 11:50, leaving its window 11:00–12:00 and its tokens unchanged.
  Its expected rows gain the time cells: "All runs" `| burler | 3 | 3000000 | 0 | 0 | 0 | 15.0M | 20.0% | 3h20m0s | 83.3% | $60.00 | claude-opus-5-5: 3 |\n| orch | | | | | | 60.0M | 80.0% | | | $240.00 | |\n| idle | | | | | | | | 40m0s | 16.7% | | |\n` and `run-a` `| burler | 1 | 1000000 | 0 | 0 | 0 | 5.0M | 33.3% | 2h0m0s | 100.0% | $20.00 | claude-opus-5-5: 1 |\n| orch | | | | | | 10.0M | 66.7% | | | $40.00 | |\n| idle | | | | | | | | 0s | 0.0% | | |\n`.
  The orch fork `agent-b`, charged wholly to `run-b` because its first prompt names it, gains two user lines after its existing ones, at 11:20 and 11:40, inside `run-b`'s idle gap; user lines carry no usage and are not its first prompt, so no charge, message count or session count moves.
  A new assertion reads `run-b`'s idle with `timeCells` as `40m0s` `66.7%`: the orch message at 11:30 charged half to `run-b` and the wholly charged fork's 11:20–11:40 lines both fall in that gap and leave it whole, which fails if either orch transcript reached the run's union.
- `TestCountRun`'s `| webster+sub | 3 | 10 | 0 | 18 | 4 | 0.0M | 70.5% |` substring still matches, since the new columns follow `share`.
- `cmd/lyx/testdata/test-budget.yaml`'s `tools/tokencount` row rises from 12 to 13 for `TestRunTime`.

**ImpactSummary:** `writeTable` has two callers, both in `Report.WriteMarkdown`, updated here; the table's column change breaks only `TestCountOrch`'s full-row strings, rewritten here.

**Commit:** 2: tokencount: working time per role and idle per run
