# Card 3 — tokencount-working-intervals

**Create:**
- `tools/tokencount/timing.go`
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#turnStamp` -> `type turnStamp struct`
- `plan:tools/tokencount#interruptMarker` -> `const interruptMarker = "[Request interrupted by user"`
- `plan:tools/tokencount#localCommandOutputPrefix` -> `const localCommandOutputPrefix = "<local-command-stdout>"`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(stamps []turnStamp) []Interval`
- `plan:tools/tokencount#unionIntervals` -> `func unionIntervals(sets ...[]Interval) []Interval`
- `plan:tools/tokencount#clipIntervals` -> `func clipIntervals(intervals []Interval, start, end time.Time) []Interval`
- `plan:tools/tokencount#totalDuration` -> `func totalDuration(intervals []Interval) time.Duration`
- `tools/tokencount/timing_test.go`
- `plan:tools/tokencount#TestCountRunWorkingTime` -> `func TestCountRunWorkingTime(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#userTexts`
- `tools/tokencount#CountRun`

**Intent:** Collect each role's working time while `countFile` walks the transcripts, per Decisions 1 and 2 of the decision record.
`line` gains a `StopReason string` field under `Message`, tagged `stop_reason`.
`Interval` is a half-open `[Start, End)` pair of `time.Time`.
`turnStamp` is one user or assistant line's parsed timestamp and whether that line ends the turn.
`endsTurn(l)` is true for an assistant line whose `StopReason` is non-empty and not `tool_use`, and for a user line whose first text, per `userTexts`, starts with `interruptMarker` or with `localCommandOutputPrefix`.
`workingIntervals(stamps)` sorts the stamps by time and returns, for each consecutive pair, the interval between them when the earlier stamp does not end the turn;
the interval after a turn-ending stamp is left out, and a stamp with no successor contributes nothing, so a session that crashes mid-turn counts its gap up to its next line as working.
`unionIntervals` merges any number of interval sets into one sorted, non-overlapping set, treating touching intervals as one;
`clipIntervals` keeps the parts of a set inside `[start, end)`, and `totalDuration` sums a set's lengths.
`RunTally` gains `Working map[string][]Interval`, the merged working intervals of every transcript counted under each role, `<role>+sub` included as its own key, unclipped;
`CountRun` initializes the map beside `Roles`.
`countFile` collects a `turnStamp` for every user or assistant line whose `Timestamp` parses as RFC 3339, in the same pass that already calls `observeTime`, and after the walk merges `workingIntervals` of that transcript into `run.Working[role]` with `unionIntervals`.
The new file `timing.go` holds the types and functions above, with a header comment saying it measures working time from turn boundaries.
`TestCountRunWorkingTime` in the new `timing_test.go` drives `CountRun` over fixture transcripts written with `writeLines` and asserts `run.Working` exactly, per role: two burler sessions whose turns overlap count their union, not their sum;
a turn that ends with an `end_turn` assistant line followed by a long gap leaves the gap out while both turns count;
a user line holding the interrupt marker ends the turn, so the gap after it is left out;
a `<local-command-stdout>` user line adds no working time after it;
a `tool_use` assistant line followed by its tool result inside the turn counts the wait as working;
an assistant line with no `stop_reason` followed by a later line counts the gap;
and a sub-agent transcript under a webster session counts under `webster+sub`, not `webster`.
Raise the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 12 to 13 for the new test.

**ImpactSummary:** Adds a Working map to RunTally and a stop_reason field to line; the token tallies, forks and windows countFile already records are unchanged.
