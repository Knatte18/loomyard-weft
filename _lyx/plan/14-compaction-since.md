# Card 14 — Shuttle reads whether a session compacted since a time

**Edit:**
- `internal/shuttleengine/engine.go`
- `internal/shuttleengine/session.go`
- `internal/shuttleengine/claudeengine/usage.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/claudeengine/usage_test.go`

**Create:**
- `plan:internal/shuttleengine#Runner.CompactedSince` -> `func (r *Runner) CompactedSince(turnEnd Event, since time.Time) (time.Time, bool, error)`

**Uses:**
- `internal/shuttleengine#SessionCycler`
- `internal/shuttleengine#ContextReading`

**Intent:**
Give the orch watcher the one reading D7's auto-compaction path needs: whether the session's transcript holds a compaction boundary newer than a given time.
`ContextTokens` cannot answer it, because the latest reading after an auto-compaction is usually the next turn's usage, not the boundary.

`SessionCycler` gains `CompactedSince(turnEnd Event, since time.Time) (at time.Time, found bool)`: the newest main-chain compaction boundary after `since` in the transcript `turnEnd` names.
`Runner.CompactedSince` delegates to it through `sessionCycler`, as `ContextTokens` does.
Claude's implementation in `usage.go` reuses `readContext`'s backward, doubling-window walk and its boundary parsing, and stops at the first boundary found or at the first entry whose timestamp is not after `since`, so a long transcript still costs reads proportional to what is newer than `since`.
Like `ContextTokens` it degrades to not found on every failure and never errors.

Tests: Claude's reader finds a boundary after `since`, ignores one at or before it, finds none in a transcript without one, and degrades on a missing or unreadable transcript; `Runner.CompactedSince` delegates and errors naming `SessionCycler` on an engine without it.

**ImpactSummary:** one added `SessionCycler` method with its Claude implementation and Runner delegate; `SessionCycler`'s only implementer is `*Claude`, and the test fakes gain the method.

**Commit:** 14: shuttle: read whether a session compacted since a given time
