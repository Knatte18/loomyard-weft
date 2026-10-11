# Card 5 — Confirm /clear and /compact by their SessionStart lines

**Create:**
- `plan:internal/shuttleengine#sessionStartedSince` -> `func sessionStartedSince(engine Engine, eventsPath string, offset int64, source SessionStartSource) bool`

**Edit:**
- `internal/shuttleengine#sendContext`
- `internal/shuttleengine#Runner.sendSessionSequence` -> `func (r *Runner) sendSessionSequence(guid string, landing SessionStartSource, sequenceFor func(SessionCycler) []PaneInput) error`
- `internal/shuttleengine#Runner.ClearSession`
- `internal/shuttleengine#Runner.ReloadPlugins`
- `internal/shuttleengine#Runner.CompactSession`
- `internal/shuttleengine#confirmPasted`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/doc.go`

**Intent:**
D6.
`sendContext` gains `landingStart SessionStartSource`, the session-start source whose line confirms a send that starts no turn; empty for every other send.
`sendSessionSequence` takes it as `landing` and sets it on the context: `ClearSession` passes `SessionStartClear`, `CompactSession` passes `SessionStartCompact`, `ReloadPlugins` passes the empty source and behaves as today.
`sendSequence` copies the context, so the field reaches `confirmPasted` unchanged.

`sessionStartedSince` reports whether the events file holds a session-start signal with source `source` past byte `offset`, the way `turnStartedSince` does for a turn start; an engine without `SessionSignalParser` and an unreadable file read as no.
In `confirmPasted`, a send with a non-empty `landingStart` counts as landed when `sessionStartedSince` finds its source past `sentAt`, checked on every read beside `turnStartedSince`.
For such a send an empty box no longer ends the reads at once: they continue, inside the same submit window and read cap, until the matching line appears;
when the window closes or the cap is reached with none, the box read decides as today, so an empty box confirms and anything else fails as today.
A box holding exactly the sent text still gets the guarded Enter, and any other text fails as today.
Bound: the wait never exceeds the existing submit window, and an engine with no session-start recording falls back to the box read at its end.

`TestRunner_ClearSession_PlaysScriptedSequence` and `TestRunner_CompactSession_PlaysSequenceForFocus` gain rows: a matching `SessionStart` line appended past the pre-send offset confirms the send; a line with the other source does not;
with none in the window the send confirms on the empty box once the window closes.
The fake engine parses the `SESSIONSTART:<source>` line card 2 introduced, and the fake reed appends it on the paste.
`shuttleengine/doc.go`'s paragraph on `Runner.ClearSession` and `Runner.CompactSession` states the confirmation.

**ImpactSummary:** `Runner.ClearSession` and `Runner.CompactSession`, and so the orch watcher's cycles, confirm on the hook line and wait up to the submit window when it never comes; `Runner.ReloadPlugins` and every other send are unchanged.
