# Card 1 — orch-compact-boundary

**Edit:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_compact_test.go`
- `internal/orchengine/doc.go`

**Uses:**
- `internal/shuttleengine/engine.go`

**Intent:**
D1: the compacting phase completes on a boundary that `CompactedSince` finds, not on the newest entry's `ContextTokens` reading.

In `Watcher.tickCompacting`, when `st.ReadingTurnEnd` is set, ask `w.session.CompactedSince(*st.ReadingTurnEnd, st.PhaseEnteredAt)`.
A found boundary qualifies whatever follows it, any `TurnEndsAfter` count included.
`CompactedSince` returns only a boundary strictly after `since`, so a boundary at or before `PhaseEnteredAt` never qualifies, as D1 states: the at-entry case flips from completing to not completing, so `TestCompact_BoundaryAtEntryCompletes` is deleted and its case becomes an assertion in `TestCompact_CompletesOnBoundaryAfterEntryAndIdle`.
On completion with the idle probe passing, as today: store `ContextTokens(*st.ReadingTurnEnd)` through `storeReading` (the transcript's newest qualifying entry, which may be a post-compaction assistant entry), increment `CycleCount`, move `CompactionBaseline` to the boundary's `At`, then `reloadAfterCompaction`.
The timeout path, the retype-when-idle path and the "no qualifying boundary read" guard keep their shape, with "qualifying" now meaning the found boundary.
Update the method's doc comment and the compacting section of `internal/orchengine/doc.go` to say the boundary is searched for rather than read off the newest entry, and why (a message answered right after the boundary hides it).

Tests in `watcher_compact_test.go`, as changes to the existing compacting tests:
- `landBoundary` stops setting the fake session's `boundary` map and sets its `autoCompact` entry for turn end `a` (the map `CompactedSince` answers from), so every existing compacting test that lands a boundary follows the new completion test;
- `TestCompact_CompletesOnBoundaryAfterEntryAndIdle` already pins the completion for a boundary with no turn end after it (reload starts, `CycleCount` increments, `CompactionBaseline` is the boundary's time, no abort) and that a boundary before the phase entry does not complete; it gains one row, a boundary followed by an assistant turn end, where `ContextTokens` reports a non-compacted reading, and the boundary exactly at the phase entry as an assertion beside the before-entry one;
- `TestCompact_RestartWithBoundaryReadTypesNothing` gains a row for a boundary followed by a turn end;
- the timeout abort past the handoff timeout stays pinned by `TestCompact_TimeoutReturnsToIdleAndHoldsAutomaticTriggers`, which needs no new case.

**ImpactSummary:** one method in `orchengine`'s watcher plus its doc; no signature changes, no callers outside the package.
