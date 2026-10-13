# Card 2 — turn-intervals

**Create:**
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#unionOf` -> `func unionOf(intervals []interval) []interval`
- `plan:tools/tokencount#clipTo` -> `func clipTo(intervals []interval, span interval) []interval`
- `plan:tools/tokencount#totalOf` -> `func totalOf(intervals []interval) time.Duration`
- `plan:tools/tokencount#turnPoint` -> `type turnPoint struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(points []turnPoint) []interval`
- `plan:tools/tokencount#RunTiming` -> `type RunTiming struct`
- `plan:tools/tokencount#RunTally.Timing` -> `func (run RunTally) Timing() RunTiming`
- `plan:tools/tokencount#timedLine` -> `func timedLine(t *testing.T, kind, ts, stopReason, text string) string`
- `plan:tools/tokencount#TestRunTime` -> `func TestRunTime(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#CountRun`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Measure each role's working time as turn time (Decisions 1 and 2) and the run's idle time (Decision 4), over the transcript window as the span for now; card 3 swaps in the archived-status span.

New file `tools/tokencount/timing.go` holds:
- `interval`, a `From, To time.Time` pair, and the helpers `unionOf` (sorts by `From` and merges overlapping or touching intervals), `clipTo` (cuts each interval to `span`, dropping empty ones) and `totalOf` (sums the lengths).
- `turnPoint`, one transcript line's `At time.Time` and `Ends bool`.
- `endsTurn`: true for an assistant line whose `Message.StopReason` is non-empty and not `tool_use`, and for a user line any of whose `userTexts` starts with `[Request interrupted by user` or with `<local-command-stdout>`.
- `workingIntervals`: sorts the points by `At` (stable) and returns, for each consecutive pair, the interval from the first to the second unless the first `Ends`.
- `RunTiming`: `Span interval`, `Roles map[string]time.Duration` and `Idle time.Duration`.
- `RunTally.Timing`: the span is `First`..`Last`; each role's time is `totalOf(clipTo(unionOf(run.Work[role]), span))`; idle is the span's length minus `totalOf(clipTo(unionOf(every role's intervals), span))`.

`line.Message` gains `StopReason string` with tag `json:"stop_reason"`.
`RunTally` gains `Work map[string][]interval`, each role's working intervals over all its transcripts, unmerged; `CountRun` initialises it beside `Roles`.
`countFile` collects a `turnPoint` for each user or assistant line with an RFC 3339 timestamp, duplicates of a counted message included, since the line's time is real, and appends `workingIntervals` of that one transcript to `run.Work[role]`, so a sub-agent transcript's intervals land under `<role>+sub` exactly where its tokens land.
The run's transcripts are only its worktree's sessions and their sub-agents; `CountOrch` reads the prime separately and never touches `Work`, so no orch interval enters a run's union.

`TestRunTime` lands in `count_test.go` as a table: each row writes one run's transcripts with `writeLines` under `projectDir(t.TempDir(), "/hub/<row>")`, calls `CountRun` then `Timing`, and asserts `Span`, every role's duration and `Idle` exactly.
`timedLine` builds a user or assistant line at `ts` with an optional `stop_reason` and a text item `text`, in the shape `endsTurn` reads.
Rows:
- two turns of one session, the first ending on an `end_turn` assistant line, then a one-hour gap before the next user line: the role holds both turns, the gap is idle;
- a user line holding `[Request interrupted by user]`: the gap after it is idle;
- a `<local-command-stdout>` user line typed between turns: no working time after it until the next line;
- a turn whose assistant line stops on `tool_use` and whose tool result arrives ten minutes later: the ten minutes count as working time;
- two sessions of one role overlapping by half: the role holds their union, not their sum;
- a session with a sub-agent transcript: the sub-agent's turn counts under `<role>+sub`, not under the role.
The package's row in `cmd/lyx/testdata/test-budget.yaml` rises by one for `TestRunTime`.

**ImpactSummary:** `countFile` gains interval collection beside its unchanged usage tally and `line` one field; token counts and every existing assertion stay as they are, and nothing renders the timing until card 4.
