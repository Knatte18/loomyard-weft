# Card 2 — Turn intervals and a run's measured time

**Create:**
- `tools/tokencount/runtime.go`
- `tools/tokencount/runtime_test.go`
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#stamp` -> `type stamp struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#turnIntervals` -> `func turnIntervals(stamps []stamp) []interval`
- `plan:tools/tokencount#mergeIntervals` -> `func mergeIntervals(in []interval) []interval`
- `plan:tools/tokencount#clipIntervals` -> `func clipIntervals(merged []interval, from, to time.Time) []interval`
- `plan:tools/tokencount#intervalsLength` -> `func intervalsLength(merged []interval) time.Duration`
- `plan:tools/tokencount#overlapLength` -> `func overlapLength(merged []interval, from, to time.Time) time.Duration`
- `plan:tools/tokencount#EntryEnd` -> `type EntryEnd struct`
- `plan:tools/tokencount#ShedRecord` -> `type ShedRecord struct`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#RunTime` -> `type RunTime struct`
- `plan:tools/tokencount#RunTally.MeasureTime` -> `func (run *RunTally) MeasureTime(rec ShedRecord)`
- `plan:tools/tokencount#timedLine` -> `func timedLine(t *testing.T, kind, ts, stopReason, text string) string`
- `plan:tools/tokencount#TestRunTime` -> `func TestRunTime(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RunTally`
- `tools/tokencount#CountRun`
- `tools/tokencount#RunTally.countFile`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decisions 1 to 6, the measurement: each transcript yields its working intervals, and a run's span, per-role time, idle and Producers split are computed from them and from the run's shed record.
Nothing renders the result yet (card 3), and nothing reads an archived record yet (cards 4 and 5): every caller still sees a zero `RunTime`.

`tools/tokencount/runtime.go` (package `main`, file comment naming what it measures) holds the new types and functions:

- `interval` is a half-open stretch `[From, To)` of wall-clock time.
- `stamp` is one transcript line's time and whether it ends a turn.
- `endsTurn(l)` is Decision 1's rule: an `assistant` line whose `Message.StopReason` is non-empty and not `tool_use`, or a `user` line any of whose `userTexts` starts with `[Request interrupted by user` or with `<local-command-stdout>`.
  Each marker is a named constant with a comment saying which Claude Code line carries it.
- `turnIntervals(stamps)` sorts the stamps by time (stable) and returns `[stamps[i].At, stamps[i+1].At)` for every consecutive pair whose first stamp does not end a turn.
  A run of stamps after the last one yields nothing, so a crashed session's gap up to its next line counts as working, per the record's auto-mode assumption.
- `mergeIntervals` returns the union as sorted, disjoint, non-empty intervals; `clipIntervals` cuts merged intervals to `[from, to)`; `intervalsLength` sums merged intervals; `overlapLength` is the length of `[from, to)` that merged intervals cover.
- `EntryEnd` is one shed history entry's producer name and end time, in recorded order.
- `ShedRecord` is what a run's archived records say about its span: `Read` (an archived status with at least one history entry was read), `Started` (the seed's `started_at`, zero when absent), `Floor` (the commit time of the slug's next-older archive tag, zero when none) and `Entries []EntryEnd`.
  Card 4 fills it from the archive tag; a zero `ShedRecord` is a run with no status.
- `ProducerTime` is one producer name's `Entries` count, `Elapsed` and `Idle`.
- `RunTime` is where a run's span went: `From`, `To`, `Roles map[string]time.Duration`, `Idle`, `FromStatus` (the span came from a shed status, false for the transcript-window fallback) and `Producers []ProducerTime` in first-appearance order.

`line`'s `Message` struct gains `StopReason string` with tag `json:"stop_reason"`; a null or absent value decodes as empty.
`RunTally` gains the unexported `work map[string][]interval` (role, `+sub` roles included, to the working intervals of its transcripts, not yet merged) and the exported `Time RunTime` that `MeasureTime` fills.
`CountRun` makes `work` alongside `Roles`.
`countFile` collects one `stamp` for every `user` or `assistant` line whose timestamp parses (`time.RFC3339Nano`, the parse `observeTime` uses), and when the walk ends it appends `turnIntervals` of them to `run.work[role]`.
Orch transcripts are read only by `CountOrch` and never reach `countFile`, so no orch session or orch sub-agent enters `work` (Decision 4).

`MeasureTime(rec)` sets `run.Time`:
- With `rec.Read` and at least one entry, the span ends at the latest `EntryEnd.End` and starts at `rec.Started`, or at `run.First` when `Started` is zero, or at the first entry's end when both are zero.
  The start is then raised to `rec.Floor` when it is earlier and lowered to the first entry's end when it is later, in that order, so it never passes the first entry (Decision 3).
  `FromStatus` is true.
- Otherwise the span is `run.First` to `run.Last` and `FromStatus` is false.
- Each role's time is `intervalsLength(clipIntervals(mergeIntervals(work[role]), From, To))`, so concurrent sessions of one role count once (Decision 2); a role with no intervals is absent from `Roles`.
- Idle is the span minus the clipped union of every role's intervals (Decision 4).
- With `FromStatus`, the Producers split walks `rec.Entries` in order with `latest` starting at `From`: each entry adds 1 to its producer's `Entries`, and when its end is later than `latest` it owns `[latest, end)`, adding that length to `Elapsed`, that length minus `overlapLength` of the run's union to `Idle`, and moving `latest` to its end (Decision 5).
  `Elapsed` over all producers therefore sums to the span.

Decision 6 needs no code: no field is added to `shedengine`'s status shape.

`TestRunTime`, in the new `tools/tokencount/runtime_test.go`, is one table test: each row writes its transcripts with `writeLines` into its own `projectDir` under `t.TempDir()`, runs `CountRun`, calls `MeasureTime` with the row's `ShedRecord`, and compares `run.Time` with the row's exact `RunTime` (`From`, `To`, every `Roles` entry, `Idle`, `FromStatus`, `Producers`).
`timedLine` builds a transcript line: kind `user` gives a user line whose content is `text`; kind `assistant` gives an assistant line with a unique message id and `stop_reason` set to `stopReason`, or `null` when `stopReason` is empty.
The rows, each with no record unless it says otherwise:
- Two turns, an `end_turn` and a long gap: user 10:00, assistant 10:05 `tool_use`, user tool result 10:06, assistant 10:10 `end_turn`, user 12:00, assistant 12:30 `end_turn`.
  The role's time is 40m, the 10:05 to 10:06 foreground tool call included, idle is 1h50m, and the span is the transcript window 10:00 to 12:30.
- The interrupt marker: user 10:00, assistant 10:01 with a null `stop_reason`, user `[Request interrupted by user]` 10:02, user 11:00, assistant 11:05 `end_turn` gives 7m.
- A local slash command: user 10:00, assistant 10:10 `end_turn`, user `<local-command-stdout>ok</local-command-stdout>` 10:20, user 10:30, assistant 10:40 `end_turn` gives 20m.
- Overlap and sub-agents: sessions titled `ly:r:burler` working 10:00 to 10:30 and `ly:r:burler-2` working 10:15 to 10:45 give `burler` 45m, not 60m.
  A `ly:r:webster` session working 10:00 to 10:10, whose sub-agent transcript works 10:20 to 10:40, gives `webster` 10m and `webster+sub` 20m.
- A status span with out-of-order entries: a record with `Read`, `Started` 09:00 and entries A 09:30, B 10:50, C 10:20 and D 11:00, over transcripts working 09:10 to 09:20 and 10:30 to 10:40.
  The span is 09:00 to 11:00 and idle is 1h40m.
  The Producers are A (1 entry, 30m elapsed, 20m idle), B (1, 1h20m, 1h10m), C (1, 0s, 0s) and D (1, 10m, 10m), summing to the 2h span.
- A seed without `started_at`, with an older session: an older session working 08:00 to 08:45 (user 08:00, assistant 08:45 `end_turn`), `Floor` 08:30 and a single entry ending 09:30 start the span at 08:30.
  The older session's role time is clipped to 15m, and idle is the remaining 45m of the 1h span, so a missing clip fails the row.
- A `started_at` before the floor: `Started` 08:00, `Floor` 08:30 and a single entry ending 09:30 start the span at 08:30.
- A start after the first entry: transcripts from 10:00, no `Started` and a first entry ending 09:30 start the span at 09:30.

The new top-level test raises the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 12 to 13.

**ImpactSummary:** additive inside `tools/tokencount`: `line` and `RunTally` gain fields, `countFile` and `CountRun` gain interval collection, and no output changes; `RunTally` literals in the calibrate and profile tests keep compiling with zero-valued new fields.

**Commit:** 2: tokencount: measure turn time, idle and the producer split per run
