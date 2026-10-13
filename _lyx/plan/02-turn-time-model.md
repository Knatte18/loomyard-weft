# Card 2 — turn-time-model

**Create:**
- `tools/tokencount/timing.go`
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#unionOf` -> `func unionOf(spans []interval) []interval`
- `plan:tools/tokencount#coveredWithin` -> `func coveredWithin(merged []interval, span interval) time.Duration`
- `plan:tools/tokencount#turnMark` -> `type turnMark struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#turnIntervals` -> `func turnIntervals(marks []turnMark) []interval`
- `plan:tools/tokencount#HistoryEnd` -> `type HistoryEnd struct`
- `plan:tools/tokencount#ShedRecord` -> `type ShedRecord struct`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#RunTally.MeasureTime` -> `func (run *RunTally) MeasureTime(record *ShedRecord)`
- `tools/tokencount/timing_test.go`
- `plan:tools/tokencount#turnLine` -> `func turnLine(t *testing.T, kind, ts, stopReason, text string) string`
- `plan:tools/tokencount#TestMeasureTime` -> `func TestMeasureTime(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#CountRun`
- `tools/tokencount#userTexts`
- `tools/tokencount#writeLines`
- `tools/tokencount#projectDir`

**Intent:**
Measure each role's working time, the run's span and idle, and the per-producer split, from transcripts and an already-decoded shed record (Decisions 1–5), with no report change and no git read yet.
`timing.go` opens with a file comment saying it measures turn time and the run's span; `timing_test.go` opens with one naming its tier (Tier-1, no git, no spawn), as `orch_test.go` does.

Turn time (Decision 1):
- `line`'s anonymous `Message` struct gains `StopReason string` tagged `json:"stop_reason"`; a JSON null and an absent key both decode as empty.
- `endsTurn` reports whether a line ends its session's turn: an assistant line whose `StopReason` is non-empty and not `tool_use`, or a user line one of whose `userTexts` starts with `[Request interrupted by user` or with `<local-command-stdout>`.
- `turnMark` holds one user or assistant line's parsed timestamp and its `endsTurn` verdict.
- `turnIntervals` sorts the marks by time, stably, and returns, for each consecutive pair, the interval from the first to the second when the first does not end the turn and the second is later; the interval after a turn-ending mark is idle, so it is never returned.
- `RunTally.countFile` collects one `turnMark` per user or assistant line whose `Timestamp` parses as RFC 3339, in the same `eachLine` pass it already makes, and after the pass appends that transcript's `turnIntervals` to the role tally's working intervals.
  A sub-agent transcript's intervals land on the `<role>+sub` tally, because `CountRun` already passes that role to `countFile`.
  A line with no parseable timestamp adds no mark, so the existing fixtures' untimed lines measure nothing.

Interval arithmetic: `interval` is a half-open `from`..`to` pair; `unionOf` sorts and merges overlapping or touching intervals; `coveredWithin` returns how much of `span` the merged intervals cover.

`RoleTally` gains an unexported `working []interval`, the raw intervals of every transcript of the role in the run, and `Time time.Duration`, the role's working time in the run.
`RunTally` gains `SpanFrom, SpanTo time.Time`, the run's span; `Idle time.Duration`; `Producers []ProducerTime`, empty when no shed status was read; and `ShedTag string`, the archive tag the shed record came from, empty when none was read.
Each new field carries a doc comment in the style of its struct's existing fields.

`HistoryEnd` is one shed history entry's `Producer` name and its `End` time, already resolved (a folded entry's `last_at`; card 4 resolves it).
`ShedRecord` is a run's decoded archived record: `Tag string`, the archive tag read; `Started time.Time`, the seed's `started_at`, zero when the seed has none; `NotBefore time.Time`, the committer time of the slug's next-older archive tag, zero when there is none; `History []HistoryEnd` in recorded order.
`ProducerTime` is one row of the Producers table: `Producer string`, `Entries int`, `Elapsed` and `Idle time.Duration`.

`MeasureTime` fills the new fields from the role tallies and `record`:
- With a nil record or one with an empty `History`, the span is the transcript window `First`..`Last`, `ShedTag` stays empty and `Producers` stays empty (Decision 3's no-status fallback).
- Otherwise the span ends at the latest `End` in `History`.
  It starts at `Started`, or at `First` when `Started` is zero; that start is raised to `NotBefore` when it is earlier, then lowered to `History[0].End` when it is later, so the lower bound never pushes the start past the first entry's end.
  A start still zero after the fallback (no seed stamp and no timed transcript line) is `History[0].End`.
  `ShedTag` is `record.Tag`.
- Each role's `Time` is how much of the span the union of its `working` intervals covers (Decision 2).
- `Idle` is the span's length less how much of it the union of every role's `working` intervals covers (Decision 4).
  Only `run.Roles` enters that union: the orch's sessions and sub-agents are counted by `CountOrch` into `OrchTally`, never into a run's roles, so they stay out and parent review counts as idle.
- With a record, each `History` entry, in order, owns the interval from the latest `End` among the entries before it (the span start for the first) to its own `End`, empty when its own `End` is not later (Decision 5).
  Per producer name, in order of first appearance, `Entries` counts its entries, `Elapsed` sums its intervals and `Idle` sums the part of them the run's working union does not cover.
  Because the start is never later than the first entry's end, the `Elapsed` values sum to the span.

`TestMeasureTime` in the new `timing_test.go` is table-driven: each row writes its transcripts into a fresh project directory with `writeLines`, runs `CountRun`, calls `MeasureTime` with the row's record, and asserts `SpanFrom`, `SpanTo`, each role's `Time`, `Idle`, `ShedTag` and `Producers` with exact equality.
`turnLine` builds one timed transcript line: a user line whose content is one text item holding `text`, or an assistant line carrying `stop_reason` set to `stopReason`, or JSON null when it is empty.
Rows, each an acceptance criterion:
- Two turns, the first holding a foreground tool call (assistant `tool_use` at 10:01, its result at 10:04), ended by `end_turn` at 10:05, then a gap to a typed prompt at 12:00 and an `end_turn` at 12:03: the role's time is 8m, the tool call's 3m included and the gap left out; no record, so the span is 10:00–12:03 and idle is 115m.
- A user line `[Request interrupted by user for tool use]` at 10:03 after a turn begun at 10:00, then a prompt at 11:00 and an `end_turn` at 11:01: the gap after the marker is idle, the role's time 4m.
- A `<local-command-stdout>` user line at 10:30 after an `end_turn` at 10:01, then a prompt at 11:00 and an `end_turn` at 11:02: the 10:30–11:00 gap adds no working time, the role's time 3m.
- Two burler sessions working 10:00–10:10 and 10:05–10:20 give burler 20m, not 25m, and a webster session's sub-agent transcript working 10:30–10:40 gives `webster+sub` 10m.
- A record with `Started` 10:00 and history entries A ending 10:05, B 10:30, A 10:25 (out of order) and C 10:50, over a burler session working 10:10–10:20: the span is 10:00–10:50, idle 40m, and the Producers are A (2 entries, 5m elapsed, 5m idle), B (1, 25m, 15m), C (1, 20m, 20m), whose elapsed sums to the span.
- A record with no `Started`, `NotBefore` 09:50 and one entry ending 10:30, over an older session working 09:00–09:10 and a burler session working 10:00–10:20: the span starts at 09:50, not 09:00, and the older session's work is clipped out of the role's time.
- A record with no `Started` and entries ending 10:30 and 11:00 over a session whose first line is at 10:40: the span starts at 10:30, the first entry's interval is empty and the second's elapsed is 30m.

Raise `tools/tokencount` in `cmd/lyx/testdata/test-budget.yaml` from 12 to 13 for `TestMeasureTime`.

**ImpactSummary:** `line`, `RoleTally` and `RunTally` gain fields every existing reader ignores, and `countFile` only records marks beside its tally, so `CountRun`, `CountOrch`, the calibration and the report are unchanged until card 3 renders the fields.
