# Card 2 — working-intervals

**Create:**
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#turnPoint` -> `type turnPoint struct`
- `plan:tools/tokencount#interruptMarker` -> `const interruptMarker = "[Request interrupted by user"`
- `plan:tools/tokencount#localCommandStdout` -> `const localCommandStdout = "<local-command-stdout>"`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(points []turnPoint) []interval`
- `plan:tools/tokencount#covered` -> `func covered(spans []interval, from, to time.Time) time.Duration`
- `plan:tools/tokencount#stoppedAssistant` -> `func stoppedAssistant(id, ts, stopReason string) string`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RunTally`
- `tools/tokencount#CountRun`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#TestCountRun`

**Uses:**
- `tools/tokencount#userTexts`
- `tools/tokencount#userPrompt`

**Intent:**
Collect each transcript's working time, turn time per Decision 1, while `countFile` walks it once.

A new file `tools/tokencount/timing.go` holds the interval code:
- `interval` is a `From, To time.Time` pair.
- `turnPoint` is one user or assistant line with a parseable timestamp: its time and whether it ends the turn.
- `endsTurn` reports a turn-ending line: an assistant line whose message's `stop_reason` is non-empty and not `tool_use`, or a user line one of whose `userTexts` starts with `interruptMarker` or `localCommandStdout`.
  A user line holding only tool results does not end the turn, so a foreground tool call, a blocking gate test included, stays working time.
- `workingIntervals` sorts the points by time (stable) and returns, for each consecutive pair, the interval between them unless the first point ends the turn.
- `covered` merges the spans and returns the length of their union inside `[from, to]`; it returns 0 when `to` is not after `from`.
  Card 3 uses it for role time, idle and producer idle.

`line.Message` gains `StopReason string` with tag `json:"stop_reason"` (a JSON null decodes as empty).
`RunTally` gains `Working map[string][]interval`, the working intervals of every transcript keyed by the role it counts under (`<role>+sub` for a sub-agent, exactly as `Roles`); `CountRun` initialises it beside `Roles`.
`countFile` appends a `turnPoint` for every user or assistant line with a parseable timestamp inside its existing `eachLine` callback, then appends `workingIntervals` of them to `run.Working[role]`; it reads no file twice.
Orch transcripts never enter: `CountRun` reads only the run's worktree project directory, and `CountOrch` keeps its own reading.

`stoppedAssistant`, a helper in `count_test.go`, is an assistant line at `ts` with message id `id`, no usage and the given `stop_reason`.
`TestCountRun` gains a fixture project `/hub/turns` built with `stoppedAssistant` and `userPrompt`, asserting `covered(run.Working[role], run.First, run.Last)`:
- `t1.jsonl`, titled `ly:turns:burler`: user 10:00; assistant 10:01 `tool_use`; tool result 10:11; assistant 10:12 `end_turn`; user 12:00; assistant 12:05 `end_turn`; a user line `<local-command-stdout>Set model</local-command-stdout>` at 12:06; user 13:00; assistant 13:02 `tool_use`; user `[Request interrupted by user]` at 13:03; user 14:00; assistant 14:01 `end_turn`.
  Its working time is 10:00–10:12 (the ten-minute tool call included), 12:00–12:05, 13:00–13:03 and 14:00–14:01: 21 minutes, with the long gaps after `end_turn`, after the local-command line and after the interrupt left out.
- `t2.jsonl`, titled `ly:turns:burler-2`: user 10:05, assistant 10:20 `end_turn`, overlapping `t1`; the burler union is 29 minutes, not the 36-minute sum.
- `t1/subagents/agent-1.jsonl`: user 10:02, assistant 10:04 `end_turn`; `burler+sub` is 2 minutes, and the burler figure stays 29 minutes.

**ImpactSummary:** `countFile` and `CountRun` gain interval collection with no change to token tallies; `line` and `RunTally` gain one field each; nothing reads `Working` until card 3.
