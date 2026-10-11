# Card 2 — working intervals

**Create:**
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#unionOf` -> `func unionOf(ivs []Interval) []Interval`
- `plan:tools/tokencount#clipTo` -> `func clipTo(ivs []Interval, span Interval) []Interval`
- `plan:tools/tokencount#totalOf` -> `func totalOf(ivs []Interval) time.Duration`
- `plan:tools/tokencount#turnClock` -> `type turnClock struct`
- `plan:tools/tokencount#turnClock.observe` -> `func (c *turnClock) observe(l line)`
- `plan:tools/tokencount#turnClock.intervals` -> `func (c *turnClock) intervals() []Interval`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#stampedLine` -> `func stampedLine(typ, ts, stopReason, text string) string`
- `tools/tokencount/worktime_test.go`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#userTexts`
- `tools/tokencount#writeLines`
- `tools/tokencount#timedAssistant`

**Intent:**
Decisions 1 and 2: working time is turn time, collected while `countFile` already walks each transcript once.
New file `worktime.go` declares `Interval` (a `Start` and `End` time), the pure helpers `unionOf` (sorted, overlapping and touching intervals merged), `clipTo` (each interval cut to a span, empty results dropped) and `totalOf` (the summed length), and the turn logic.
`turnClock.observe` takes every user or assistant line with a parseable RFC 3339 timestamp, as `observeTime` already parses them, and keeps its time and whether `endsTurn` holds; `turnClock.intervals` sorts those stable by time and returns the interval between each consecutive pair unless the first of the pair ends the turn, then returns the union.
`endsTurn` holds for an assistant line whose `stop_reason` is set and not `tool_use`, for a user line any of whose `userTexts` starts with `[Request interrupted by user`, and for a user line any of whose `userTexts` starts with `<local-command-stdout>`; a tool-result user line has no text item and never ends a turn, and a line with no `message` object is never ending.
A Webster fork's transcript repeats its parent's context lines with their original timestamps and message ids (a repeated assistant message carries the same timestamp in the fork and in its parent, as the transcripts on disk show; the repeated lines get fresh uuids, so a uuid cannot identify a copy).
`countFile` therefore keeps a run-level set of the lines earlier transcripts of the run contributed, keyed by line type, timestamp text and, for an assistant line, message id, and a line already in the set is not fed to the `turnClock`; sessions are counted before their sub-agents, so the parent's lines win, and a line is added to the set only after its own transcript is done, so a transcript never skips its own lines.
Without it a fork's `+sub` time would hold the parent's pre-fork working intervals.
A coincident timestamp between unrelated sessions of one run could drop an instant of work, a bound the millisecond resolution makes negligible.
`line`'s message struct gains `StopReason string` with the tag `stop_reason`, so a null reads as unset.
`RoleTally` gains `Work []Interval`, the union of the intervals of every transcript counted under the role, and `RunTally` gains `Work []Interval`, the union over all of the run's transcripts; `countFile` feeds a `turnClock` from the lines it already visits and merges the result into the role's and the run's `Work`.
`+sub` is its own role key already, so `<role>+sub` holds the union of that role's sub-agent transcripts, and a role's concurrent sessions union instead of sum.
`CountRun` reads only the run's own project directory, so an orch session or an orch sub-agent, which lives in the prime's directory, can never enter `Work`; Card 4's test covers that.
Nothing is clipped here, because the span is known only once Card 3's status is read; the card changes no report output.
A foreground tool call counts as working time, and a session that crashed mid-turn with no marker counts its gap to the next line as working, per the record's assumption.
`worktime_test.go` holds `TestCountRunWorkingTime`, one table over transcript sets written with `writeLines`, each row asserting the summed `totalOf` of a role's `Work`, and the helper `stampedLine`, which writes a user or assistant transcript line at a timestamp with an optional `stop_reason` and text because `timedAssistant` and `assistant` write none; Cards 4 and 6 build their transcripts with it.
`RunTally` also gains the unexported set the repeated-line rule needs.
Rows: two turns separated by an `end_turn` line and a one-hour gap count both turns and leave the gap out; a user interrupt marker line ends the turn, so the gap after it is idle; a `<local-command-stdout>` user line adds no working time after it; a `tool_use` assistant line, a long-running tool result and a closing `end_turn` line count the whole span as working; two same-role sessions that overlap count their union, not their sum; a sub-agent transcript counts under `<role>+sub` and not under its parent's role; lines written out of timestamp order and a line with an unparsable timestamp give the same total as the ordered, parsable lines; a transcript whose last assistant line carries no `stop_reason` counts its gap to the next line; a `custom-title` line and a user line with no `message` object add no interval and raise no error; a sub-agent transcript that repeats the parent's earlier lines (same timestamps and assistant message ids) holds only its own new work under `<role>+sub`, while the parent's role keeps the repeated lines' time.
The tokencount row in `test-budget.yaml` rises by one.

**ImpactSummary:** `countFile` has one caller, `CountRun`, and the new fields and `stop_reason` tag are additive, so no existing assertion or report line changes.
