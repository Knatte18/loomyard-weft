# Card 6 — Gate sends wait for an idle writer and retry instead of ending the gate

**Create:**
- `internal/shuttleengine#Run.sendWithin`
- `internal/shuttleengine#Run.boundaryIdle`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.handleGatedBoundary`
- `internal/shuttleengine/gate_test.go`
- `internal/shuttleengine/gate_pending_test.go`

**Uses:**
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#unmatchedTurnStart`
- `internal/shuttleengine#turnStartHold`
- `internal/shuttleengine#ErrSessionBusy`
- `internal/shuttleengine#ErrSubmissionNotLanded`
- `internal/shuttleengine#SessionCycler`

**Intent:**
Implement D6 in the gated wait loop.

`Run` gains three fields: a `turnStartHold` for a boundary that a `turn_start` cleared, a flag that a boundary is cleared that way, and the unsent-re-prompt mark.

`boundaryIdle` confirms the boundary idle in fact before the gate is evaluated for a send, keyed on the same capability pair as card 3's `awaitIdleSession`: for an engine with both the idle reading (`SessionCycler`) and session signals, an `unmatchedTurnStart` past the boundary's turn end clears `gateAtBoundary` and the mark and records the cleared state, and the loop keeps polling with no send and no failure.
While a boundary is cleared that way, `Wait` captures the pane every tick for D1's idle reading, which times the hold.
On the tick where the hold releases (`TurnStartInterrupt` reports the turn interrupted, or the pane has read idle on every poll for `turnStartIdleOverride`), the boundary is restored and `handleGatedBoundary` runs as on a new turn end, whether or not an entry is pending; the unsent-re-prompt mark is not restored, since that evaluation re-derives any re-prompt from the gate's recorded state.
A new turn end clears the cleared state as today's arrival does.
An engine lacking either answers idle at once, so its loop is unchanged: without the idle reading nothing could time the 10 s release, so it reads no session signals here, as the Shared Decision `idle-reading-capability` states.

`sendWithin` is the wait loop's send: `Run.Send`'s validation and `sendVerified` with `sendContext.deadline` set to `run.deadline`, so the idle wait and the submit window both end by the run deadline.
`handleGatedBoundary`'s pending send and re-prompt send go through it.
A re-prompt send that fails with `ErrSessionBusy` or `ErrSubmissionNotLanded` leaves the entry's failure and sent counts unchanged, clears the memo, keeps the writer at the boundary and sets the unsent-re-prompt mark, logging one Warn; any other re-prompt send error ends the loop as today.
`Wait`'s per-tick branch that re-enters `handleGatedBoundary` for `gatePending && gateAtBoundary` re-enters it for `unsentReprompt && gateAtBoundary` too, so the re-prompt is re-sent on a later tick with no new writer event; a writer turn that starts meanwhile clears the boundary and the mark, and its next turn end is evaluated afresh.

Bound (this card adds a retry edge): the retry spends no gate budget and is bounded by the run deadline, which still reaches `classifyDeadlineExpiry` and the gate's final evaluation.
A boundary cleared by a `turn_start` is either followed by that turn's end or restored within `turnStartIdleOverride` of idle pane.
A send that starts with less time left than its windows overshoots the deadline by at most its tails: the idle wait's last capture, the post-Enter confirm read, the clear with its readings and the final capture.
`wait.go`'s file doc comment and `handleGatedBoundary`'s doc comment describe the idle boundary, the retry and the budget.
`handleGatedBoundary`'s return statements keep their markers, so the completion-signal tripwire's audited set is unchanged.

Tests on a `fakeClock` with card 3's fake engine carrying both the idle reading and session signals: `gate_test.go` gains two new table tests, each for a behavior no existing test covers.
`TestGate_WaitsForIdleWriter` pins an `AttachGated`-style replay over a writer mid-turn typing no gate send until a turn end follows the writer's newest `turn_start`, with rows for that turn end, the release by `TurnStartInterrupt`, and 10 s of idle pane restoring the boundary and evaluating the gate.
`TestGate_UnsentRepromptRetries` pins the retry, with rows where a busy re-prompt and an unlanded re-prompt each spend no attempt and are re-sent on a later tick, the unlanded one into the box its failed send cleared; where a writer turn starting meanwhile cancels the pending re-send; and where a re-prompt starting with less time left than its windows ends its idle wait, and in a second row its submit window, by the run deadline.
`TestGate_SendFailsMidLoop_EndsLoopWithAttemptsSoFar`, whose send fails with an error that is neither sentinel, keeps pinning that such an error ends the loop and is left unchanged.
`gate_pending_test.go`'s pending-send rows are adjusted only where the boundary check now reads the pane.

**ImpactSummary:** Confined to the gated branch of `Run.Wait` and `handleGatedBoundary`; an ungated run and an engine without both the idle reading and session signals loop exactly as before.

**Commit:** 6: shuttle: gate sends wait for an idle writer and retry an unlanded re-prompt
