# Card 2 — working intervals

**Create:**
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#turnTracker` -> `type turnTracker struct`
- `plan:tools/tokencount#turnTracker.observe` -> `func (t *turnTracker) observe(l line)`
- `plan:tools/tokencount#turnTracker.intervals` -> `func (t *turnTracker) intervals() []Interval`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#unionIntervals` -> `func unionIntervals(sets ...[]Interval) []Interval`
- `plan:tools/tokencount#clipIntervals` -> `func clipIntervals(in []Interval, span Interval) []Interval`
- `plan:tools/tokencount#intervalsTotal` -> `func intervalsTotal(in []Interval) time.Duration`
- `plan:tools/tokencount#stampedAssistant` -> `func stampedAssistant(ts, stopReason string) string`
- `plan:tools/tokencount#stampedUser` -> `func stampedUser(ts, contentJSON string) string`
- `plan:tools/tokencount#TestWorkingIntervals` -> `func TestWorkingIntervals(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** The run's working time is collected where `countFile` already walks every transcript once.
`Interval` is a start and an end `time.Time`, declared with the helpers in a new file `tools/tokencount/intervals.go`; the tracker, `endsTurn` and the three interval functions live there too.
`line` gains the assistant message's `stop_reason` in its anonymous `message` struct, a struct field, so it takes no arrow.
`RunTally` gains the field `Work map[string][]Interval`, keyed like `Roles` by role, `+sub` included; `countFile` feeds each line to a fresh `turnTracker` and appends the transcript's intervals to `run.Work[role]`, creating the map entry on first use.
The tracker keeps every user or assistant line whose timestamp parses, as RFC 3339 like `observeTime`; `intervals` orders them by timestamp, stably, and returns the interval between each line and the next, except where the first line ends the turn.
`endsTurn` is true for an assistant line whose `stop_reason` is set and is not `tool_use`, for a user line whose first text, from `userTexts`, starts with `[Request interrupted by user`, and for a user line whose first text starts with `<local-command-stdout>`.
A tool result user line has no text item, so it never ends a turn, and a foreground tool call counts as working time.
The last line of a transcript opens no interval, so a session that crashed mid-turn counts its time up to its last line and nothing after.
`unionIntervals` merges any number of interval sets into sorted, non-overlapping intervals; `clipIntervals` cuts intervals to a span and drops the empty ones; `intervalsTotal` sums durations.
Nothing here adds a flag, a verb or a guard, and `Work` is read by cards 4 and 5 only.
`TestWorkingIntervals` is one new top-level test, a table of synthetic transcripts counted through `CountRun`, each row asserting `intervalsTotal(unionIntervals(run.Work[role]))` or the merged intervals, and its rows are:
- two turns each closed by `end_turn`, an hour apart, count both turns and leave the gap out;
- a user line with the interrupt marker ends the turn, so the gap after it is idle;
- a local slash command's `<local-command-stdout>` user line adds no working time after it;
- an assistant `tool_use` line, its tool result and a closing `end_turn` line count as one working interval;
- an assistant line with no `stop_reason`, a line whose timestamp does not parse, and a transcript's last line open no idle gap and add no time after the last line;
- two sessions of one role whose intervals overlap count their union, and a sub-agent transcript under that session lands in `Work["burler+sub"]`.
The test and the two helpers sit in `count_test.go`, and the test runs with `t.Parallel()`.
It uses the existing `writeLines`, `toolResult` and `jsonString` helpers, and the two new helpers `stampedAssistant` and `stampedUser`, which build timestamped transcript lines with a `stop_reason` and a content value.
The new top-level test takes the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 12 to 13.

**ImpactSummary:** `line` and `RunTally` gain one field each, so no composite literal of either changes meaning; `countFile` has one caller, `CountRun`, and still tallies tokens as before.
