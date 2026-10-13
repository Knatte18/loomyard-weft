# Card 2 — Turn time per role and the idle row

**Create:**
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#Interval.Duration` -> `func (i Interval) Duration() time.Duration`
- `plan:tools/tokencount#unionIntervals` -> `func unionIntervals(spans []Interval) []Interval`
- `plan:tools/tokencount#coveredWithin` -> `func coveredWithin(spans []Interval, within Interval) time.Duration`
- `plan:tools/tokencount#turnMark` -> `type turnMark struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(marks []turnMark) []Interval`
- `plan:tools/tokencount#RunTally.settleTimes` -> `func (run *RunTally) settleTimes()`
- `plan:tools/tokencount#interruptMarker` -> `const interruptMarker = "[Request interrupted by user"`
- `plan:tools/tokencount#localCommandStdout` -> `const localCommandStdout = "<local-command-stdout>"`
- `plan:tools/tokencount#timeRule` -> `const timeRule = "Time is turn time: a session works from each of its user or assistant lines to its next one, except after an assistant line that ends its turn, an interrupt or a local command's output. A role's time is the union of its sessions' working time within the run's span, so concurrent sessions of one role count once, and time shares can sum past 100% where roles overlap, as a +sub row overlaps its parent. The idle row is the span minus the union of every session's working time: time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push. The union of working time plus idle is the span; the orch's sessions enter no union, so their time is idle."`
- `plan:tools/tokencount#userAt` -> `func userAt(t *testing.T, ts, text string) string`
- `plan:tools/tokencount#assistantAt` -> `func assistantAt(id, ts, stopReason string) string`
- `plan:tools/tokencount#TestTurnTime` -> `func TestTurnTime(t *testing.T)`

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

**Uses:**
- `tools/tokencount#userTexts`
- `tools/tokencount#eachLine`
- `tools/tokencount#CountOrch`
- `tools/tokencount#writeLines`
- `tools/tokencount#projectDir`
- `tools/tokencount#userPrompt`
- `tools/tokencount#jsonString`

**Intent:**
Decisions 1, 2, 4 and 7, with the transcript window as each run's span; card 3 replaces the span with the archived status's.
The interval code lives in a new file `tools/tokencount/turntime.go` with a one-line file comment.

Decision 1, working time is turn time.
`line.Message` gains `StopReason string` with JSON key `stop_reason` (a JSON null decodes as empty).
`endsTurn(l)` is true for an assistant line whose `StopReason` is non-empty and not `tool_use`, and for a user line any of whose `userTexts` starts with `interruptMarker` or `localCommandStdout`.
`turnMark` holds a line's parsed time and its `endsTurn` value.
`countFile` collects a `turnMark` for every user or assistant line whose timestamp parses as RFC 3339, the lines `observeTime` already reads, and after the walk sorts them stably by time and calls `workingIntervals`: each pair of consecutive marks is one interval, working unless its first mark ends the turn.
A foreground tool call is working time, because its `tool_use` assistant line does not end the turn.
`countFile` appends the transcript's intervals to the role's `RoleTally.Working` and to `RunTally.Working`, the run's every working interval; a sub-agent transcript is counted under `<role>+sub` exactly as its tokens are, so it lands in that row's `Working`.

Decision 2 and 4, union and clip.
`Interval` is `Start, End time.Time`, and `Duration` is `End - Start`, zero when `End` is not after `Start`.
`unionIntervals` sorts and merges overlapping or touching intervals; `coveredWithin` is the total duration of the union of `spans` clipped to `within`.
`RunTally` gains `Span Interval`, the run's span, and `Idle time.Duration`; `RoleTally` gains `Working []Interval` and `Time time.Duration`.
`settleTimes` sets each role's `Time` to `coveredWithin(role.Working, run.Span)` and `run.Idle` to `run.Span.Duration() - coveredWithin(run.Working, run.Span)`.
`CountRun` sets `Span` to `First..Last` and calls `settleTimes` before it returns.
Only `CountRun`'s worktree transcripts enter `Working`; `CountOrch` reads the prime's orch sessions into `OrchTally` and never touches a `RunTally`, so orch time, an orch sub-agent charged wholly to the run included, stays idle.

Decision 7, the table.
`writeTable` takes the table's span and idle, and its header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`.
A role row's time cell is `t.Time.Round(time.Second)` printed with `%s`, and its time share is `100 * Time / span` printed `%.1f%%`, `0.0%` for a zero span.
The orch row gets two blank cells for time and time share.
After the role rows (and the orch row) comes `| idle | | | | | | | | <idle> | <idle share> | | |`, both cells formatted like a role's.
Rows keep their weight order and `share` keeps its meaning.
`WriteMarkdown` passes each run's `Span.Duration()` and `Idle`, and for "All runs" sums each role's `Time`, the runs' `Idle` and the runs' span durations, taking no union across runs; it prints `timeRule` as its own paragraph right after the "All runs" table's total-weight line.

Decision 10: `run`'s report title becomes `# Token usage and time by role`; the output path stays.
The package doc in `main.go` gains a paragraph after the share-column paragraph describing the `time` and `time share` columns, the `idle` row and how each is measured, per this card, and its first sentence says the report holds token use and time.

Tests:
- `TestTurnTime` is table-driven, each row a set of transcripts written with `writeLines` under one run's project directory and counted with `CountRun`, asserting each named role's `Time`, the run's `Idle`, and rendered rows of `Report{Runs: ...}.WriteMarkdown`.
  Rows: two turns of a burler session split by an `end_turn` line and an hour's gap count both turns and leave the gap idle, and the first turn holds a `tool_use` assistant line followed later by its tool result, whose gap counts as working; a user line `[Request interrupted by user]` after a null-stop assistant line makes the gap to the next prompt idle; a `<local-command-stdout>` user line after an `end_turn` adds no working time up to the next prompt; two burler sessions whose turns overlap count their union, and a sub-agent transcript under one of them counts as `burler+sub`; an orch session in a separate prime project directory whose sub-agent's prompt names the run (built with `userPrompt`) and whose messages fall inside the run's idle gap is counted by `CountOrch`, and the report rendered with that `OrchTally` still prints the run's idle row with the whole gap, which fails if orch time ever enters the run's union.
  The orch row's blank time cells are pinned by `TestCountOrch`, below, not here.
  `userAt` writes a user line at a timestamp with string content, and `assistantAt` an assistant line at a timestamp with a message id, one output token of usage and the given `stop_reason`, `null` when empty.
  After the rows, one report over every row's run asserts the "All runs" idle row holds the sum of the rows' idles and the sum-of-spans share, and the report holds `timeRule`.
- `TestCountOrch`'s two full-row assertions gain the new cells: the "All runs" burler row `| 4h0m0s | 100.0% |` after `20.0% |` and run-a's `| 2h0m0s | 100.0% |` after `33.3% |`, and each orch row two blank cells after its share; its fixture's untimed-stop lines leave every gap working.
  `TestCountRun`'s and `TestEstimatedCost`'s row substrings stop before or start after the new cells and stay as they are.
- `cmd/lyx/testdata/test-budget.yaml`'s `tools/tokencount` row goes from 12 to 13 for `TestTurnTime`.

**ImpactSummary:** Local to `tools/tokencount`: `writeTable`'s only caller is `Report.WriteMarkdown`, the role tables gain three cells per row and an idle row, `TestCountOrch` is the one test asserting a full row, and the budget row rises by one.

**Commit:** 2: tokencount reports turn time per role and the run's idle time
