# Card 2 — tokencount-working-intervals

**Create:**
- `plan:tools/tokencount#Interval` -> `type Interval struct`
- `plan:tools/tokencount#mergeIntervals` -> `func mergeIntervals(in []Interval) []Interval`
- `plan:tools/tokencount#turnPoint` -> `type turnPoint struct`
- `plan:tools/tokencount#turnPointOf` -> `func turnPointOf(l line) (turnPoint, bool)`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#workIntervals` -> `func workIntervals(points []turnPoint) []Interval`
- `plan:tools/tokencount#turnLine` -> `func turnLine(typ, ts, stop, text string) string`
- `plan:tools/tokencount#TestCountRunWorkTime` -> `func TestCountRunWorkTime(t *testing.T)`

**Edit:**
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#line`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#CountRun`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Working time is turn time (Decision 1), and this card collects it per transcript; nothing reads it yet.
A new file `tools/tokencount/interval.go` declares `Interval` (`Start`, `End` as `time.Time`), `mergeIntervals` (sorted, disjoint, touching intervals joined, empty ones dropped) and the turn logic.
`line.Message` gains a `StopReason string` field read from `stop_reason`, where a JSON null decodes to the empty string.
`turnPointOf` returns a transcript line's `turnPoint{At, EndsTurn}` for a `user` or `assistant` line whose `timestamp` parses as `time.RFC3339Nano`, and false otherwise.
`endsTurn` is true for an assistant line whose `StopReason` is non-empty and not `tool_use`, for a user line with a text item (through `userTexts`) starting with `[Request interrupted by user` and for one starting with `<local-command-stdout>`; a user line holding only tool results ends nothing.
`workIntervals` stably sorts the points by `At` and returns one interval per consecutive pair, `[points[i].At, points[i+1].At]`, unless `points[i].EndsTurn`; the last point opens no interval.
`RunTally.countFile` collects the points while it walks the file, beside `observeTime` and before the `seen` message-id check so a repeated message's turn-ending line still counts, with `eachLine`'s error now returned after the walk instead of directly, and appends the transcript's intervals to the `Work` of the `RoleTally` it counts under and to the run's own `Work`.
`CountRun` merges every role's `Work` and the run's `Work` once all transcripts are counted, so a role's `Work` is the union of its transcripts' intervals (Decision 2) and the run's `Work` is the union over every role and `+sub` row of the run's own worktree transcripts (Decision 4); `CountRun` reads only that worktree's project directory, so the orch session never enters either.
`RoleTally.Work` and `RunTally.Work` are new `[]Interval` fields, and `RoleTally` values built for the "All runs" sum leave `Work` nil.
Intervals are not clipped here; the span is not known until a later card.
A foreground tool call keeps its turn open because the assistant line before it has `stop_reason` `tool_use`, so the wait for the tool result counts as working time.
`TestCountRunWorkTime` is one table over synthetic transcripts written with `writeLines`, `toolResult` and the new `turnLine` helper (`typ` is `user` or `assistant`; `stop` is the assistant line's `stop_reason`, empty for null; `text` is the user line's string content), each case asserting the exact `Work` of the roles and of the run:
- two turns separated by an `end_turn` and an hour's gap give two intervals and leave the gap out, and a line without a timestamp is ignored;
- a `tool_use` assistant line, a tool result ten minutes later and an `end_turn` give one interval across the ten minutes;
- a user line `[Request interrupted by user for tool use]` ends the turn, and the gap after it is idle;
- a user line `<local-command-stdout>ok</local-command-stdout>` after an `end_turn` adds no working time after that line;
- two sessions titled `ly:x:burler` and `ly:x:burler-2` with overlapping turns give one merged interval, not their sum, and a webster session with one sub-agent transcript gives separate `webster` and `webster+sub` intervals whose union is the run's `Work`;
- lines written out of timestamp order are ordered by timestamp.
The branch where a transcript ends mid-turn with no interrupt marker is the Decision's crash assumption and is not a separate branch: the gap after a non-ending line counts as working in every case above.
`cmd/lyx/testdata/test-budget.yaml` raises the `tools/tokencount` row by one for `TestCountRunWorkTime`.

**ImpactSummary:** `RoleTally`, `RunTally` and `line` only gain fields, so every composite literal and decoder in the package keeps compiling and its output is unchanged; `countFile` and `CountRun` keep their signatures and token counts, and `CountOrch` neither reads nor writes `Work`.
