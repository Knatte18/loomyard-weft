# Card 4 — a run's span, role times and idle

**Create:**
- `plan:tools/tokencount#RunTime` -> `type RunTime struct`
- `plan:tools/tokencount#RunTally.Measure` -> `func (run *RunTally) Measure(archive *RunArchive)`
- `plan:tools/tokencount#spanOf` -> `func spanOf(run RunTally, archive *RunArchive) (Interval, bool)`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(e shedengine.HistoryEntry) (time.Time, bool)`
- `plan:tools/tokencount#clipIntervals` -> `func clipIntervals(in []Interval, window Interval) []Interval`
- `plan:tools/tokencount#totalOf` -> `func totalOf(in []Interval) time.Duration`
- `plan:tools/tokencount#TestRunTimeMeasure` -> `func TestRunTimeMeasure(t *testing.T)`

**Edit:**
- `tools/tokencount#RunTally`
- `tools/tokencount#CountRun`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#RunArchive`
- `plan:tools/tokencount#Interval`
- `plan:tools/tokencount#mergeIntervals`

**Intent:** With the working intervals per role and the archived records in hand, this card computes a run's span, each role's time inside it and the idle remainder; no report prints them yet.
The new symbols go in `tools/tokencount/worktime.go`, which card 3 created, and the new imports are `internal/shedengine` for `HistoryEntry` and `internal/shedrun` for `Seed`'s fields.

`RunTime` holds `Window Interval` (the span's start and end, zero for a sum over runs), `Span time.Duration`, `Roles map[string]time.Duration` (a role's time keyed as `Roles` of `RunTally` is, `<role>+sub` rows included), `Idle time.Duration` and `StatusRead bool`.
`RunTally` gains `Archive *RunArchive` and `Time RunTime`, and its doc comment, "one task run's usage, split by role", gains the time.
`Measure` stores `archive` in `run.Archive`, takes the span from `spanOf`, sets each role's time to the total of its `Work` clipped to the span, and sets `Idle` to the span minus the total of the union of every role's clipped `Work`.
The union covers the run's worktree transcripts and their sub-agents, because those are the only transcripts `CountRun` reads; the orch session and every orch sub-agent are never in a role's `Work`, so parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push all land in `Idle`.
`CountRun` calls `run.Measure(nil)` as its last step, so a run counted without records still has a `Time` measured against its transcript window, and a later `Measure(archive)` replaces it.

`spanOf` returns the span and whether a shed status was read.
With a nil archive, or a status none of whose history entries has a parseable end, it returns `run.First` to `run.Last` and false.
Otherwise the end is the latest `entryEnd` among the entries: `LastAt` when the entry is folded (`Repeats` > 0) and `LastAt` parses, else `At`, both parsed as RFC 3339; an entry whose end does not parse is skipped, here and in the first-entry bound below, and the report never aborts for it.
The start is the seed's `StartedAt` when it parses, else `run.First`; it is then raised to `archive.OlderTagAt` when that is later, and lowered to the end of the first entry in recorded order that has an end when it is later than that or when it is zero.
Both bounds apply to either start.

`clipIntervals` returns the parts of the sorted, disjoint input inside the window; `totalOf` sums the lengths of intervals.

`TestRunTimeMeasure` is one table over `Measure` on literal `RunTally` values (a `Roles` map of `RoleTally` with `Work`, and `First`/`Last`) and literal `RunArchive` values with a `shedrun.Seed` and `shedengine.Status` history, asserting `Window`, `Span`, each role's time, `Idle` and `StatusRead`; it needs no transcript file and no git.
Rows, on 2026-01-02: no archive, where the window is the transcript window, two roles overlap and idle is the window minus their union; a status whose last entry ends after the transcripts, where the span runs from the seed's `started_at` to that end and idle counts the waits before and after the sessions; a role interval running past the latest entry's end, which is clipped out of the role's time; a folded entry, where the end is its `last_at`; entries recorded out of order, the first recorded ending at 11:00, a later-recorded one at 10:00 and the seed's `started_at` at 10:45, where the end is the latest and the start stays 10:45 because the first-entry bound is the first recorded entry's end, not the earliest; a seed without `started_at`, where the span starts at the earliest transcript line; the same seed-less start with an older tag's time later than that line, which raises the start, and with a first entry ending before that line, which lowers it; a reused slug, where an older tag's time later than the seed's `started_at` raises the start and a role interval before it is clipped out of the role's time; a first entry ending before the transcripts' first line and before `started_at`, where the start is lowered to that end; an entry with an `at` that does not parse, which owns no end, so the span ends at the latest parseable one; an empty history, which gives the transcript window with `StatusRead` false.
The skipped-entry branch is reached by the unparseable-`at` row; the empty-history branch by the empty-history row.

The `tools/tokencount` row in `test-budget.yaml` goes from 15 to 16.

**ImpactSummary:** `RunTally` gains two fields and `CountRun` one call, and `CountRun` now runs `Measure` for every run, so every caller sees a non-zero `Time` and no existing test reads it; the budget row moves by one.
