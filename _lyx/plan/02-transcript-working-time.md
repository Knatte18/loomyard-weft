# Card 2 — transcript-working-time

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#Interval.Len` -> `func (i Interval) Len() time.Duration`
- `plan:tools/tokencount#turnLine` -> `type turnLine struct`
- `plan:tools/tokencount#interruptMarker` -> `const interruptMarker = "[Request interrupted by user"`
- `plan:tools/tokencount#localCommandStdout` -> `const localCommandStdout = "<local-command-stdout>"`
- `plan:tools/tokencount#turnEnds` -> `func turnEnds(l line) bool`
- `plan:tools/tokencount#workingIntervals` -> `func workingIntervals(lines []turnLine) []Interval`
- `plan:tools/tokencount#union` -> `func union(intervals []Interval) []Interval`
- `plan:tools/tokencount#clip` -> `func clip(intervals []Interval, span Interval) []Interval`
- `plan:tools/tokencount#total` -> `func total(intervals []Interval) time.Duration`
- `plan:tools/tokencount#TestCountRunWorkingTime` -> `func TestCountRunWorkingTime(t *testing.T)`

**Uses:**
- `tools/tokencount#userTexts`
- `tools/tokencount#RunTally.observeTime`

**Intent:** Measure each transcript's working time as turn time and collect it per role and per run, Decisions 1 and 2, in a new file `tools/tokencount/timing.go` that holds the interval types and helpers.
`Interval` is a closed `Start`..`End` pair of `time.Time` with `Len`.
`turnLine` is one timestamped user or assistant line of a transcript: its time and whether it ends the turn.
`line.Message` gains `StopReason string` from the `stop_reason` key.
`turnEnds` is true for an assistant line whose `StopReason` is set and is not `tool_use`, and for a user line whose text, the join of `userTexts`, starts with `interruptMarker` or with `localCommandStdout`; every other user or assistant line continues the turn.
`workingIntervals` orders the lines by time, stable, and returns one interval per consecutive pair whose first line does not end the turn; the gap after a turn-ending line is idle, and nothing follows the last line.
`union` merges overlapping and touching intervals into a sorted disjoint set; `clip` keeps the parts inside `span`; `total` sums lengths.
`RoleTally` gains `Working []Interval`, the union of its transcripts' working intervals; `RunTally` gains `Working []Interval`, the union over every transcript `CountRun` reads, sub-agents included.
`countFile` collects the transcript's `turnLine`s from the lines it already walks, timestamps parsed as `observeTime` parses them, then unions `workingIntervals` of them into the role's and the run's `Working`.
Orch transcripts are read by `CountOrch`, never by `countFile`, so they enter no union (Decision 4).
A session that crashes mid-turn leaves no turn-ending line, so its gap until its next line counts as working, the auto-mode assumption.
`TestCountRunWorkingTime`, in `count_test.go`, builds synthetic transcripts and asserts the exact `Working` intervals per role and for the run: two turns of one burler separated by an `end_turn` assistant line and a long gap count both turns and not the gap; a `tool_use` assistant line followed by a tool result and an `end_turn` line counts the foreground call; a user line opening with the interrupt marker ends the turn, so the gap after it is left out; a `<local-command-stdout>` user line between turns adds nothing after it; two burler sessions whose turns overlap count their union once; a webster sub-agent's turn lands under `webster+sub`, not `webster`; a line without a timestamp or one that does not parse is ignored.
One new top-level test, so the `tools/tokencount` row in `cmd/lyx/testdata/test-budget.yaml` rises by one.

**ImpactSummary:** Three structs gain a field each and `countFile` gains a collection step; every existing caller and `TestCountRun` keep passing since no signature changes.
