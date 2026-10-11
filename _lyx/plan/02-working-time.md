# Card 2 — Count each role's working intervals from turn boundaries

**Create:**
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#turnMark` -> `type turnMark struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workIntervals` -> `func workIntervals(marks []turnMark) []interval`
- `plan:tools/tokencount#unionOf` -> `func unionOf(spans []interval) []interval`
- `plan:tools/tokencount#lengthOf` -> `func lengthOf(spans []interval) time.Duration`
- `plan:tools/tokencount#turnLine` -> `func turnLine(t *testing.T, kind, ts, stopReason, text string) string`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#TestCountRun`

**Intent:**
Decisions 1 and 2, the collection half: each transcript yields its working intervals, and each role keeps the union of its transcripts' intervals, `<role>+sub` its sub-agents' as tokens count them.
Clipping to the run's span and the idle row come in later cards; this card records the raw union.

`line`'s `Message` gains `StopReason string` with tag `json:"stop_reason"` (a JSON null decodes as empty).

A new file `tools/tokencount/worktime.go` (package `main`, file comment saying it measures working time from turn boundaries) holds:
- `interval`, a half-open `[Start, End)` pair of `time.Time`.
- `turnMark`, one user or assistant line's parsed timestamp and whether it ends a turn.
- `endsTurn(l)`: true for an assistant line whose `StopReason` is non-empty and not `"tool_use"`, and for a user line any of whose `userTexts` starts with `[Request interrupted by user` or with `<local-command-stdout>`; false otherwise.
- `workIntervals(marks)`: stable-sorts the marks by time and returns, for each consecutive pair, the interval from the first to the second unless the first ends a turn; an empty interval (equal times) is dropped.
  A transcript that crashes mid-turn, with no closing line, counts its gap to its next line as working, per the record's auto-mode assumption.
- `unionOf(spans)`: the sorted union, overlapping and touching intervals merged.
- `lengthOf(spans)`: the summed length of already-disjoint intervals.

`RoleTally` gains an unexported field `work []interval`, the union of its transcripts' working intervals.
`RunTally.countFile` collects a `turnMark` for every user or assistant line whose timestamp parses with `time.RFC3339Nano` (other line types and unparseable timestamps are skipped, as `observeTime` skips them), and after the file's lines sets `tally.work = unionOf(append(tally.work, workIntervals(marks)...))`.
Because `CountRun` already passes `role+"+sub"` for sub-agent transcripts, `+sub` rows get their own union and the parent row holds none of the sub-agents' time.
A foreground tool call sits between an assistant `tool_use` line and its user result line, so it counts as working time with no extra rule.

Test: `TestCountRun` gains a block over a fresh project dir `/hub/times`, built with the new helper `turnLine(t, kind, ts, stopReason, text)` in `count_test.go`, which renders a `user` line whose content is the string `text`, or an `assistant` line with `stop_reason` set to `stopReason` (JSON null when empty) and a unique message id, at timestamp `ts`.
The block asserts `lengthOf(run.Roles[role].work)` exactly per role:
- `burler`, session one: user 10:00, assistant 10:05 `tool_use`, user 10:20 (the tool result, the foreground tool call), assistant 10:30 `end_turn`, user 12:00, assistant 12:10 `end_turn`: 40m, the 10:30–12:00 gap left out.
- `burler`, session two (title `burler-2`): user 10:20, assistant 10:40 `end_turn`, overlapping session one: the row is 50m, the union, not the 60m sum.
- `burler+sub`, a sub-agent of session one: user 10:01, assistant 10:03 `end_turn`: 2m, and `burler` stays 50m.
- `writer`: user 13:00, assistant 13:05 `tool_use`, user 13:06 `[Request interrupted by user for tool use]`, user 14:00, assistant 14:10 `end_turn`: 16m, the gap after the interrupt marker left out.
- `reviewer`: user 15:00, assistant 15:10 `end_turn`, user 15:30 `<local-command-stdout>Set model</local-command-stdout>`, user 16:00, assistant 16:05 `end_turn`: 15m; without the local-command rule the 15:30–16:00 gap would add 30m.

**ImpactSummary:** `line` and `RoleTally` gain one field each and `countFile` one extra pass over collected marks; token counts and every existing report row are unchanged.

**Commit:** 2: tokencount: working intervals per role from turn boundaries
