# Card 2 — Working intervals from transcript turns

**Create:**
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#turnEvent` -> `type turnEvent struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(events []turnEvent) []Interval`
- `plan:tools/tokencount#union` -> `func union(spans []Interval) []Interval`
- `plan:tools/tokencount#covered` -> `func covered(spans []Interval, from, to time.Time) time.Duration`
- `plan:tools/tokencount#TestTurnTime` -> `func TestTurnTime(t *testing.T)`
- `plan:tools/tokencount#turnLine` -> `func turnLine(id, ts, stopReason string) string`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decisions 1 and 2: `CountRun` records, per transcript, the intervals in which the session is in a turn, keyed by role, so later cards can measure each role's time and the run's idle time.

`line.Message` gains `StopReason string` with JSON key `stop_reason` (a struct field, no arrow).
`Interval` is `From, To time.Time`, half-open.
`turnEvent` is one user or assistant line's parsed timestamp and whether it ends the turn.
`endsTurn` reports a turn-ending line: an assistant line whose `StopReason` is non-empty and not `tool_use`; a user line one of whose `userTexts` starts with `[Request interrupted by user`; a user line one of whose `userTexts` starts with `<local-command-stdout>`.
`workingIntervals` sorts the events by timestamp (stable) and returns, for each consecutive pair, the interval from the first to the second unless the first ends the turn; zero-length intervals are dropped.
`union` returns the sorted, merged union of its intervals; `covered` returns the length of the union of `spans` that lies within `[from, to)`.
These live in a new file `tools/tokencount/span.go` with a one-line file comment.

`RunTally` gains `Working map[string][]Interval`, keyed by role exactly as `Roles` is, so a sub-agent transcript lands under `<role>+sub`, and `Busy []Interval`, every transcript's working intervals in the run.
`countFile` collects a `turnEvent` for each user or assistant line with a parseable RFC 3339 timestamp while it already walks the file, then appends `workingIntervals` of them to `Working[role]` and to `Busy`.
Only `CountRun` calls `countFile`, so orch transcripts, counted by `CountOrch` over the prime's project directory, never enter `Busy` (Decision 4).
A crashed session's gap to its next line counts as working, per the record's auto-mode assumption; nothing special handles it.
`CountRun` initialises `Working`; the existing `RunTally` literals in the calibrate and profile tests leave it nil, which reads as no working time.

The existing transcript helpers write no `stop_reason`, and `assistant` writes no timestamp, so a new test helper `turnLine` in `count_test.go` writes an assistant line at `ts` with a usage block and `stop_reason` set to `stopReason`, omitted when empty; user lines are written inline as `TestCountOrch` writes them.

New top-level `TestTurnTime` in `count_test.go`, table-driven over synthetic transcripts written with `writeLines` and `turnLine` into a temp projects directory and counted by `CountRun`, asserts the exact `Interval` lists of `run.Working[role]` and `run.Busy`, so a wrong boundary fails the row; the overlapping-sessions row asserts the exact intervals of `union(run.Working[role])`:
- two turns, the first ending on an assistant `end_turn` line followed by a one-hour gap: both turns count, the gap does not;
- a user `[Request interrupted by user]` line: the gap after it is not counted;
- a `<local-command-stdout>` user line between turns: no time after it is counted;
- an assistant line with `stop_reason` `tool_use` followed ten minutes later by its tool result: the ten minutes count;
- two overlapping sessions of one role: their union counts, not their sum;
- a sub-agent transcript under `<session>/subagents/`: its time is under `<role>+sub`, not `<role>`, and in `Busy`.
The `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` rises by one.

**ImpactSummary:** additive fields on `line` and `RunTally` and new collection inside `countFile`; token counts, fork tallies and every existing test assertion stay as they are.

**Commit:** 2: tokencount: working intervals from transcript turns
