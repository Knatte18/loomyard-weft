# Card 1 — shed-paused-reasons

**Edit:**
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#StepResult`
- `internal/shedengine#TestStep_PauseRequested`
- `internal/shedengine#TestStep_CancelledContextWithProducerError`

**Intent:**
D13's shed half: every `paused` step stop carries a reason, so the Go driver can tell the designed pause paths from an unexplained one.

In `Shed.stepLocked`, the step-3 branch `if st.PauseRequested || ctx.Err() != nil` returns `Reason: "cancelled"` when `ctx.Err() != nil` and `Reason: "pause requested"` otherwise.
The `case callErr != nil && ctx.Err() != nil` branch after the producer call returns `Reason: "cancelled"`.
Declare the two words as unexported constants beside `conditionReason`, never as condition reasons: `isConditionReason` in `internal/shedverbs` keeps reading only `paused before ` and `paused after ` prefixes, so the loop still classifies these stops as `halted`, not `stop-condition`.
Nothing persisted changes: the status file still carries no reason, and history, `pause_requested` and the conditions are written exactly as today.
The doc comment of `StepResult.Reason` adds the two paused reasons beside the condition reason.

Tests, rows only:
- `TestStep_PauseRequested` asserts `res.Reason == "pause requested"`, and gains a second case with a context cancelled before `Step` and no `pause_requested`, asserting `Reason == "cancelled"`, `producer.calls == 0` and `State == StatePaused`, which reaches the step-3 branch's cancelled arm.
- `TestStep_CancelledContextWithProducerError` asserts `res.Reason == "cancelled"`, which reaches the after-call branch.
`step_equivalence_test.go` already compares `Run`'s reason with the final step's, so a run paused through either path stays equal there.

**ImpactSummary:** one method and one doc comment in `internal/shedengine`; the reason reaches the step envelope's `reason` key and no reader branches on an empty paused reason today.
