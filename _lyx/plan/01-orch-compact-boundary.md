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
`CompactedSince` returns only a boundary strictly after `since`, so a boundary at or before `PhaseEnteredAt` never qualifies, as D1 states: the at-entry case flips from completing to not completing, and `TestCompact_BoundaryAtEntryCompletes` is rewritten to assert that (renamed to say so).
On completion with the idle probe passing, as today: store `ContextTokens(*st.ReadingTurnEnd)` through `storeReading` (the transcript's newest qualifying entry, which may be a post-compaction assistant entry), increment `CycleCount`, move `CompactionBaseline` to the boundary's `At`, then `reloadAfterCompaction`.
The timeout path, the retype-when-idle path and the "no qualifying boundary read" guard keep their shape, with "qualifying" now meaning the found boundary.
Update the method's doc comment and the compacting section of `internal/orchengine/doc.go` to say the boundary is searched for rather than read off the newest entry, and why (a message answered right after the boundary hides it).

Tests in `watcher_compact_test.go`, extending the existing compacting tests and the fake session's `autoCompact` map where they already cover the surface:
- a boundary after the phase entry followed by an assistant turn end (so `ContextTokens` reports a non-compacted reading) completes on the next tick with the idle probe passing: reload starts, `cycle_count` increments, `CompactionBaseline` is the boundary's time, no `compaction timed out` abort;
- a boundary with no turn end after it completes the same way;
- a boundary before the phase entry does not complete, and the timeout abort still fires past the handoff timeout;
- a watcher restarted in `compacting` after a post-boundary turn end completes without typing `/compact` again.

**ImpactSummary:** one method in `orchengine`'s watcher plus its doc; no signature changes, no callers outside the package.
