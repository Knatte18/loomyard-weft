# Card 2 — turn-time

**Create:**
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#union` -> `func union(spans []Interval) []Interval`
- `plan:tools/tokencount#clip` -> `func clip(spans []Interval, window Interval) []Interval`
- `plan:tools/tokencount#length` -> `func length(spans []Interval) time.Duration`
- `plan:tools/tokencount#turnStamp` -> `type turnStamp struct`
- `plan:tools/tokencount#turnIntervals` -> `func turnIntervals(stamps []turnStamp) []Interval`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#RunTally.RoleTime` -> `func (run RunTally) RoleTime(role string) time.Duration`
- `plan:tools/tokencount#RunTally.Idle` -> `func (run RunTally) Idle() time.Duration`
- `plan:tools/tokencount#tableTime` -> `type tableTime struct`
- `plan:tools/tokencount#TestRunTime` -> `func TestRunTime(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#CountRun`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, times tableTime)`
- `tools/tokencount#TestCountOrch`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#userTexts`
- `tools/tokencount#eachLine`
- `tools/tokencount#CountOrch`

**Intent:** Measure each role's working time as turn time (Decisions 1, 2, 4 and 7 of the decision record) and print it beside the tokens, with an `idle` row per role table.
Until card 3 reads archived status, every run's span is its transcript window.

New file `tools/tokencount/worktime.go` holds the interval arithmetic and the turn rule:
- `Interval` is a `Start`/`End` pair of `time.Time`.
- `union` returns the spans sorted by start with overlapping or touching spans merged.
- `clip` intersects each span with `window` and drops the empty ones.
- `length` sums `End - Start` over spans the caller has already merged.
- `turnStamp` is one transcript line's parsed timestamp and whether `endsTurn` holds for it.
- `turnIntervals` sorts the stamps by time, stably, and returns the interval between each stamp and the next unless the earlier stamp ends a turn; a zero-length interval is dropped.
- `endsTurn` holds for an assistant line whose `stop_reason` is non-empty and not `tool_use`, and for a user line one of whose texts (`userTexts`: the string content or a text item) starts with `[Request interrupted by user` or with `<local-command-stdout>`.
  Every other line keeps the session in its turn, so a foreground tool call, a blocking gate test included, counts as working time.

`line`'s `Message` gains `StopReason string` decoded from `stop_reason`, where a JSON null decodes as empty.
`countFile` collects a `turnStamp` for every user or assistant line whose timestamp parses as RFC 3339, and after the walk appends `turnIntervals` of that one transcript to the role tally's new `Working []Interval` field.
A sub-agent transcript lands under its `<role>+sub` tally exactly as its tokens do.
`RunTally` gains `Span Interval`; `CountRun` sets it to `First`..`Last` before returning.
`RoleTime` is `length(clip(union(<role's Working>), run.Span))`, zero for a role the run lacks, so concurrent sessions of one role count once.
`Idle` is the span's length minus `length(clip(union(<every role's Working>), run.Span))`.
Orch transcripts never enter a run's tallies, since `CountOrch` reads them separately, so the orch's time, an orch sub-agent charged wholly to the run included, lands in the run's idle.

`tableTime` carries a table's per-role time, idle and span.
`writeTable` adds `time` and `time share` columns after `share`: header `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`.
A role row's time cell is its duration rounded to the second (`time.Duration`'s own format), and its time share is the duration over the table's span, as `%.1f%%`, `0.0%` for a zero span.
Rows keep their weight order; the orch row's two time cells are blank; a final `idle` row carries blank token, weight, share, cost and model cells and the idle time and its time share.
`WriteMarkdown` builds the "All runs" `tableTime` by summing each run's `RoleTime` per role, its `Idle` and its span length, with no union across runs, and each run's table from that run alone.
Under each run's heading, before its table, a line gives the span and says it is the transcript window because no shed status was read.
Below the "All runs" table, one paragraph states once: time is turn time, from a session's prompt to the end of its turn; idle is the part of the span in which no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push; role time shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent; and the union of working time plus idle is the span.

`TestCountOrch`'s two full-row assertions for the `burler` and `orch` rows gain the new time cells; its other assertions keep their text.
`TestRunTime`, in a new `worktime_test.go`, builds synthetic transcripts with `writeLines` and asserts exact `RoleTime`, `Idle` and report rows:
- two turns separated by an `end_turn` assistant line and a long gap count both turns and leave the gap out;
- a user line holding the interrupt marker, as string content and as a text item, ends the turn;
- a `<local-command-stdout>` user line typed between turns adds no working time after it;
- a `tool_use` assistant line followed by a tool result after a long gap counts the gap;
- two overlapping sessions of one role count their union, and a sub-agent transcript counts under `<role>+sub`;
- an orch session and an orch sub-agent naming the run, counted by `CountOrch` beside the run, leave the run's role times unchanged and fall into its idle;
- the report carries the new header, the `idle` row, blank orch time cells, and "All runs" time cells that sum two runs' times, idle and spans.

`cmd/lyx/testdata/test-budget.yaml`: raise the `tools/tokencount` row by the one test this card adds.

**ImpactSummary:** Every role table of the report gains two columns and an `idle` row, and each run section gains a span line; token figures, weights, shares and the other sections are unchanged.

**Commit:** 2: tokencount: role working time as turn time, idle row
