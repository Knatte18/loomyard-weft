# Card 2 — Role tables gain turn time, time share and an idle row

**Create:**
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#Interval.Duration` -> `func (i Interval) Duration() time.Duration`
- `plan:tools/tokencount#unionIntervals` -> `func unionIntervals(set []Interval) []Interval`
- `plan:tools/tokencount#covered` -> `func covered(set []Interval, window Interval) time.Duration`
- `plan:tools/tokencount#turnPoint` -> `type turnPoint struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(points []turnPoint) []Interval`
- `plan:tools/tokencount#RunTally.RoleTime` -> `func (run RunTally) RoleTime(role string) time.Duration`
- `plan:tools/tokencount#RunTally.Idle` -> `func (run RunTally) Idle() time.Duration`
- `plan:tools/tokencount#tableTime` -> `type tableTime struct`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#CountRun`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, times tableTime, orch *OrchCharge)`
- `tools/tokencount/count_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#userTexts`
- `tools/tokencount#eachLine`
- `tools/tokencount#RunTally.observeTime`

**Intent:**
Measure each role's working time as turn time and print it beside the tokens, with an `idle` row per role table (decision record, Decisions 1, 2, 4 and 7).
This card's span is the run's transcript window, `First`..`Last`; card 4 replaces it with the span read from the run's archived shed status.

`Interval` is a `From`/`To` pair of `time.Time`, and `Duration` is `To - From`, 0 when `To` is not after `From`.
`unionIntervals` sorts a set by `From` and merges overlapping and touching intervals, dropping empty ones.
`covered` returns how much of `window` the union of `set` covers.

`line`'s `Message` gains `StopReason string` with JSON key `stop_reason`; a null decodes as empty.
`endsTurn` reports whether a line ends its session's turn: an assistant line whose `StopReason` is non-empty and not `tool_use`, or a user line one of whose `userTexts` starts with `[Request interrupted by user` or with `<local-command-stdout>`.
`turnPoint` is one user or assistant line's parsed timestamp and its `endsTurn` answer.
`workingIntervals` orders a transcript's points by timestamp, stably, and returns the interval from each point to the next, except the interval after a point that ends the turn.
A foreground tool call sits between an assistant `tool_use` line and its user tool-result line, so it counts as working time; a session that crashes mid-turn counts its gap to its next line as working (decision record, Auto-mode assumptions).

`countFile` collects one `turnPoint` per user or assistant line with a parseable RFC 3339 timestamp, the lines `observeTime` already reads, and after the walk adds the transcript's `workingIntervals` to the role tally's new `Working` field and to the run's new `Working` field, each kept as a union through `unionIntervals`.
So a role's `Working` is the union over its transcripts, a sub-agent's transcripts union under `<role>+sub`, and the run's `Working` is the union over every transcript `CountRun` reads, the run's worktree sessions and their sub-agents.
Orch sessions are counted by `CountOrch` from the prime's project directory and never pass through `countFile` of a run, so no orch transcript enters a run's union and parent review counts as idle.

`RunTally` gains `Span Interval`, which `CountRun` sets to `First`..`Last` after its walk.
`RoleTime` returns `covered(roles[role].Working, run.Span)`, 0 for a role the run lacks.
`Idle` returns `run.Span.Duration()` minus `covered(run.Working, run.Span)`.

`tableTime` holds one table's figures: a duration per role, the idle duration and the span duration.
`WriteMarkdown` builds one per run from `RoleTime`, `Idle` and `Span.Duration()`, and the "All runs" one by summing each run's figures per role, idle and span, with no union across runs.
`writeTable` prints two columns after `share`, `time` and `time share`: the duration rounded to the second in `time.Duration`'s format, and its percentage of the table's span in the `%.1f%%` form `share` uses, `0.0%` when the span is 0.
Rows keep their weight order.
Below the role rows and above the orch row it prints an `idle` row with the idle time and time share and blank token, weight, share, cost and models cells; the orch row's `time` and `time share` cells are blank.
Below the "All runs" table, after the total-weight line, `WriteMarkdown` prints one paragraph stating that time is turn time, a session working from a user or assistant line until the line that ends its turn; that idle is time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push; that time shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent; and that the union of working time plus idle is the span.

Tests: a new test `TestRunTimeByRole` in `count_test.go` over `CountRun` and `WriteMarkdown` on synthetic, timestamped transcripts, with exact durations:
- a burler session with two turns, each closed by an `end_turn` assistant line and separated by a long gap, counts both turns and not the gap;
- a user line holding `[Request interrupted by user]` ends the turn, and the gap after it is idle;
- a `<local-command-stdout>` user line typed between turns adds no working time after it;
- an assistant `tool_use` line followed after a long gap by its tool result counts the gap as working;
- two overlapping sessions of one role count their union, and a sub-agent transcript counts under `<role>+sub`;
- `Idle` is the span minus the union of all working intervals, and the report's role table carries the `time` and `time share` cells, the `idle` row and, with an `OrchTally` charge given, an orch row whose time cells are blank;
- two runs whose working intervals overlap in wall-clock time: "All runs" gives each role the sum of its per-run times and idle the sum of the per-run idle, divides time share by the sum of the runs' spans, and is followed by the explanatory paragraph.

The existing `TestCountRun` assertion on the `webster+sub` row is a prefix that ends at the `share` cell, so it still holds.
Raise `tools/tokencount`'s row in `cmd/lyx/testdata/test-budget.yaml` by the new top-level test.

**ImpactSummary:** Every role table gains two columns and an `idle` row, and the report gains one explanatory paragraph; token, weight, share and cost figures are unchanged, and `writeTable`'s only caller is `Report.WriteMarkdown`, on this card.
