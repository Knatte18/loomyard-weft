# Card 5 — shuttle's pre-write guard: operator activity and an unreadable box

**Create:**
- `plan:internal/shuttleengine#operatorActivityHold` -> `func operatorActivityHold(reed ReedOps, guid string) string`
- `plan:internal/shuttleengine#readinessReasonNoInputBox` -> `const readinessReasonNoInputBox = "no input box is on screen; a menu or picker may be open"`

**Edit:**
- `internal/shuttleengine#boxReading`
- `internal/shuttleengine#readBox`
- `internal/shuttleengine#judgeReadiness`
- `internal/shuttleengine#busyReading`
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#Runner.SessionIdle`
- `internal/shuttleengine#ErrSessionBusy`
- `internal/shuttleengine#TestSend_WaitsForIdleSession`
- `internal/shuttleengine#TestRunner_SessionIdle_ReturnsScriptedClassification`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#ReedOps`

**Intent:**
Make the reading every write waits on hold for an active operator and for a box it cannot read (decisions D3 and D4).

D4: `boxReading` gains `reader bool`, which `readBox` sets when the engine implements `InputBoxReader`.
`judgeReadiness` holds a reading with `box.reader && !box.read` with `readinessReasonNoInputBox`, checked after the too-short hold and before the draft hold, so a picker or menu that shows no box is never typed into.
An engine without `InputBoxReader` reads `reader` false and is never held on that ground.
On the unknown fallback the engine's `IdleSession` reading already requires an on-screen empty box, so both paths hold an unreadable box.

D3: `operatorActivityHold(reed, guid)` calls `reed.OperatorActive(guid)` once and returns `"the operator typed in this pane within the last N s"` (N the returned quiet window in whole seconds) when active, and "" otherwise.
A query error logs at Warn ("shuttle: operator activity unreadable, not holding the write", with the strand and the error) and returns "", so a tmux that cannot answer `list-clients` stalls no run (A4); the empty-box hold still stands.
`busyReading` asks it as its last step, only when the rest of the reading passed, so every idle-wait poll takes one fresh reading of readiness, box and activity immediately before the caller types.
`awaitIdleSession`'s branch for an engine without the `SessionCycler` idle reading keeps `requireReadyAgentPane` and then asks `operatorActivityHold` once, failing at once with an error wrapping `ErrSessionBusy` that names the reason; that branch has no idle wait to poll in.
`Runner.SessionIdle` applies the same hold after a ready reading (`IdleProbe{Reason: …}`), so the orch watcher's probe holds an active operator with its reason instead of entering a send that would wait it out.
`ErrSessionBusy`'s doc comment adds the operator-activity and no-input-box reasons.

Bound: the activity check covers the last `operator_quiet_s` before each reading, not keys pressed in the milliseconds between the reading and the write (R2), and mouse or resize activity holds the same way (R4).

Tests:
- `TestSend_WaitsForIdleSession` rows: a fake reed whose `ActiveQueue` reads active for the first polls then not types nothing while active and goes through after, with no `paste:` or send entry before the release; active through the whole window fails with an error wrapping `ErrSessionBusy` whose text names "the operator typed in this pane within the last 5 s"; an `ActivityErr` with a drafted box still holds on the draft, and with an empty box goes through.
- `TestRunner_SessionIdle_ReturnsScriptedClassification` rows: `readinessEngine` (an `InputBoxReader`) over a capture its `InputBoxText` reads as no box answers not idle with `readinessReasonNoInputBox`; `idleEngine` (no `InputBoxReader`) over the same capture is not held on that ground; an active operator answers not idle with the activity reason.

Update the verified-send section of `internal/shuttleengine/doc.go` with the pre-write guard.

**ImpactSummary:** The readiness reading and the idle wait gain two holds; every verified send, `SessionIdle` and the orch probe see them, nothing else changes.
