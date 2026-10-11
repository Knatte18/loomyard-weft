# Card 3 — working intervals per role

**Create:**
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#turnLine` -> `type turnLine struct`
- `plan:tools/tokencount#turnOf` -> `func turnOf(l line) (turnLine, bool)`
- `plan:tools/tokencount#workIntervals` -> `func workIntervals(lines []turnLine) []Interval`
- `plan:tools/tokencount#mergeIntervals` -> `func mergeIntervals(in []Interval) []Interval`
- `plan:tools/tokencount#assistantLine` -> `func assistantLine(id, ts, stopReason string) string`
- `plan:tools/tokencount#TestRoleWorkingTime` -> `func TestRoleWorkingTime(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#CountRun`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#userPrompt`
- `tools/tokencount#toolResult`

**Intent:** `countFile` already walks every transcript once, and this card makes that walk also collect each transcript's working intervals, so a role's working time exists as data before any report reads it.
The new production symbols go in a new file `tools/tokencount/worktime.go`, package `main`.

`line`'s message gains `StopReason *string` tagged `stop_reason`, a pointer so `null` and an absent key both read as unset.
`RoleTally` gains `Work []Interval`: the union of the working intervals of every transcript counted under the role, unclipped, sorted and disjoint once `CountRun` returns; `<role>+sub` is its own `RoleTally`, so it has its own `Work`.
`Interval` is a half-open span with `Start` and `End`.

`turnOf` returns a user or assistant line's parsed timestamp (`time.RFC3339Nano`) and whether the line ends the turn, and reports false for any other line type and for an unparseable timestamp.
A line ends the turn when it is an assistant line whose `StopReason` is set and not `tool_use`; or a user line one of whose `userTexts` starts with `[Request interrupted by user`; or a user line one of whose texts starts with `<local-command-stdout>`.
A user line holding only a tool result has no text and never ends a turn.
`workIntervals` stable-sorts the lines by time and returns one interval between each consecutive pair, from the first line's time to the second's, except when the first line ends the turn; a zero-length interval is dropped.
`mergeIntervals` returns the sorted union of its input, overlapping and touching intervals joined.

`countFile` appends a `turnLine` per line `turnOf` accepts, next to its `observeTime` call and before the repeated-message-id early return, so the second line of one API message, which may be the only one carrying `stop_reason`, is never lost; once the transcript is read it appends `workIntervals` of them to the counted role's `Work`.
The doc comments of `RoleTally` and `RunTally` are updated to name the working intervals and, from the next card, the time.
`CountRun` replaces each role's `Work` with `mergeIntervals` of it after the last session, so concurrent sessions of one role count once.
Orch transcripts are read by `CountOrch`, never by `CountRun`, so they enter no role's `Work`; no other change is needed for the orch exclusion, and `TestReportTime` in a later card pins it at the report.

What this can let through: a crashed session that left no interrupt marker counts the gap to its next line as working, and an assistant line whose `stop_reason` is `null` on a turn's last line extends the turn to the next line.
Both are the record's named assumption and open risk; the only bound is the clip to the run's span, which a later card applies.

`TestRoleWorkingTime` is one table over `CountRun` on synthetic project directories, comparing `Roles[role].Work` with `Interval.Start`/`End` equality; each row gives its transcript files, relative to the project directory, with a `custom-title` line first.
Rows, on 2026-01-02: two turns, each ended by an `end_turn` assistant line, with a two-hour gap between them, giving the two turns' intervals and not the gap; a `[Request interrupted by user for tool use]` user line ending the turn, with a later typed prompt, giving no time between them; a `<local-command-stdout>` user line after an `end_turn` followed ten minutes later by a typed prompt, giving no time between the stdout line and the prompt; an assistant `tool_use` line, a tool result twenty minutes later and an `end_turn` line, giving one continuous interval, which is the foreground tool call; a turn with no ending line and a typed prompt two hours later, giving the gap as working; an assistant line with `stop_reason` null as the last line before a later prompt, giving the gap as working; lines written out of timestamp order, giving the intervals of the sorted order; one message id on two assistant lines, the second carrying `end_turn`, giving the turn ended at the second line; an attachment line with a timestamp, a `custom-title` line and a user line with a timestamp that does not parse, all inside an ended turn's gap, giving no interval from them; two sessions of one role overlapping in time, giving their union, with a sub-agent transcript under one of them giving a separate `<role>+sub` union.
The helper `assistantLine` builds the timed assistant line with a `stop_reason`, or `null` when the argument is empty, which the existing `assistant` and `timedAssistant` helpers cannot; it carries a one-token `usage` and takes a message id, so the repeated-id row reaches the early return that skips a usage line already counted; the typed-prompt lines use the existing `userPrompt` and the tool-result row the existing `toolResult`.

The `tools/tokencount` row in `test-budget.yaml` goes from 14 to 15.

**ImpactSummary:** `line` and `RoleTally` gain a field and `countFile` and `CountRun` gain collection steps; no caller changes, the existing tests assert only usage and the unchanged first columns, and the budget row moves by one.
