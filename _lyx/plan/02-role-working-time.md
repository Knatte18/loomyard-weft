# Card 2 — Per-role working time, idle and time columns

**Create:**
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#turnMark` -> `type turnMark struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workIntervals` -> `func workIntervals(marks []turnMark) []interval`
- `plan:tools/tokencount#unionLength` -> `func unionLength(spans []interval, from, to time.Time) time.Duration`
- `plan:tools/tokencount#RunTally.Span` -> `func (run RunTally) Span() time.Duration`
- `plan:tools/tokencount#RunTally.RoleTimes` -> `func (run RunTally) RoleTimes() map[string]time.Duration`
- `plan:tools/tokencount#RunTally.Idle` -> `func (run RunTally) Idle() time.Duration`
- `plan:tools/tokencount#formatDuration` -> `func formatDuration(d time.Duration) string`
- `plan:tools/tokencount#turnLine` -> `func turnLine(t *testing.T, kind, ts, stopReason, text string) string`
- `plan:tools/tokencount#TestRunTime` -> `func TestRunTime(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#CountRun`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, times map[string]time.Duration, idle, span time.Duration)`
- `tools/tokencount#TestCountOrch`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decisions 1, 2, 4 and 7, with Decision 3's no-status span: each role table reports the time each role's sessions spend in a turn and the time no session of the run does.

Turn time (Decision 1).
`line.Message` gains `StopReason string` with tag `json:"stop_reason"`; a null decodes as empty.
`endsTurn` reports whether a line ends its session's turn: an assistant line whose `StopReason` is non-empty and not `tool_use`, or a user line one of whose `userTexts` starts with `[Request interrupted by user` or with `<local-command-stdout>`.
`turnMark` holds one user or assistant line's parsed timestamp and its `endsTurn` answer; a line of another type, or whose timestamp does not parse with `time.RFC3339Nano`, gives no mark.
`workIntervals` sorts one transcript's marks by time (stable, so equal times keep file order) and returns, for each pair of consecutive marks, the interval from the first to the second, unless the first ends the turn; a zero-length interval is dropped.
So a foreground tool call, an assistant line with `stop_reason` `tool_use` followed by its result, counts as working, and the gap after an `end_turn`, an interrupt or a local command's output is idle until the next line.
A session that crashes mid-turn leaves no turn-ending line, so its gap until its next line counts as working (the record's auto-mode assumption).

Collection.
`RunTally` gains an unexported `work map[string][]interval` keyed by role, the `+sub` roles included, and `SpanFrom, SpanTo time.Time`, the run's span.
`countFile` collects its transcript's marks in the `eachLine` pass it already makes and, after the pass, appends `workIntervals` of them to `run.work[role]`, so a `<role>+sub` transcript lands under `<role>+sub` exactly as its tokens do.
`CountRun` sets `SpanFrom, SpanTo` to the run's `First, Last` after counting, the transcript window; card 3 replaces it with the archived span where one is read.

Time (Decisions 2 and 4).
`unionLength` merges the intervals, clips the union to `[from, to]` and returns its length, so overlapping sessions count once.
`Span` returns `SpanTo - SpanFrom`, zero when either is zero.
`RoleTimes` maps each role in `run.work` to `unionLength` of its intervals over the span.
`Idle` is `Span` minus `unionLength` of every role's intervals together.
Only the run's own project directory's transcripts and their sub-agents enter `run.work`; `CountOrch` reads the prime's directory into its own tally, and `Report.WriteMarkdown` takes a run's idle from `Idle` alone, so no orch session or orch sub-agent, even one charged wholly to the run, enters a union, and the run's idle holds that time.

Report (Decision 7).
`writeTable` takes the table's role times, idle and span.
Its header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`, `time` and `time share` after `share`, and rows keep their weight order.
`time` prints `formatDuration`, the duration rounded to the second in `time.Duration`'s `String` form; `time share` is `%.1f%%` of the table's span, and `0.0%` when the span is zero.
A role with no entry in the times map prints `0s`.
Below the roles comes the `idle` row, `| idle | | | | | | | | <time> | <time share> | | |`, then the orch row, whose `time` and `time share` cells are blank as all its token cells but weight, share and cost already are.
`Report.WriteMarkdown` passes each run's `RoleTimes`, `Idle` and `Span` to its table, and for "All runs" sums each run's role times by role, its idle and its span, with no union across runs.
Below the "All runs" table it states, once, that a role's time is the union of its sessions' turns, from a user or assistant line to the next one unless the first ends the turn; that idle is the span's time in which no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push; that time shares divide by the table's span and can sum past 100% where sessions overlap, a `+sub` row overlapping its parent; and that the union of working time plus idle is the span.
Below each run's table it prints `Span <formatDuration> from <SpanFrom> to <SpanTo>, the transcript window; no shed status was read.`, both instants in the local `2006-01-02 15:04` form the Orchestrator section uses.

Tests.
`TestRunTime`, in `count_test.go`, is table-driven over fixture transcripts written with `writeLines` and `turnLine`, which renders a user or assistant line of the given kind at `ts` with a `stop_reason` (omitted when empty) and, for a user line, the text item `text`.
Each row runs `CountRun` and asserts `RoleTimes`, `Idle` and `Span` exactly:
- two turns of one session separated by an `end_turn` assistant line and a one-hour gap count both turns and leave the gap out, which is the run's idle;
- a user line holding `[Request interrupted by user]` ends the turn: the gap after it is idle;
- a `<local-command-stdout>` user line typed between turns adds no working time after it;
- an assistant `tool_use` line followed by its tool result 30 minutes later counts the 30 minutes;
- two overlapping sessions of one role count their union, and a sub-agent transcript of the role counts under `<role>+sub` and overlaps its parent;
- a run with a one-hour gap between turns, beside a prime directory holding an orch session and an orch sub-agent whose prompt names the run and whose messages fall inside that gap, renders `Report{Runs: ..., Orch: &orch}.WriteMarkdown` after `CountOrch`, and the run's `idle` row shows the whole gap and its share of the span, while the orch row carries the sub-agent's charge; a report that folded orch time into the run's union would print a smaller idle.
A last row renders `Report{Runs: ...}.WriteMarkdown` over two counted runs and asserts the header line, a role row's time and time share cells, the `idle` row, the "All runs" table's summed time, idle and time share over the summed spans, the statement that time shares can sum past 100% where sessions overlap and that working time plus idle is the span, and the per-run span line.
`TestCountOrch`'s exact table rows gain the time and time share cells, its orch rows two blank cells, and it asserts the orch rows' time cells are blank.
`TestCountRun`'s `| webster+sub | ... | 70.5% |` assertion is a prefix and stays true.
The package's top-level tests rise by one, so its `tools/tokencount` row in `cmd/lyx/testdata/test-budget.yaml` goes from 12 to 13.

**ImpactSummary:** confined to `tools/tokencount`: `writeTable`'s one caller is `Report.WriteMarkdown`, the role tables of every report gain two columns and an `idle` row, and `TestCountOrch` is the one existing test asserting whole table rows.

**Commit:** 2: tokencount: per-role working time, idle and time columns
