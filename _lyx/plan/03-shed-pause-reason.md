# Card 3 — shed-pause-reason

**Edit:**
- `internal/shedengine#Status`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Shed.persist`
- `internal/shedengine#Shed.persistTransient` -> `func (s *Shed) persistTransient(nextCurrentProducer string, nextState State, nextError string, nextHistory []HistoryEntry, consumePause, clearConditions bool, routedTo string, transient TransientClass, parentNotice, pauseReason string) error`
- `internal/shedengine#Shed.persistMarked` -> `func (s *Shed) persistMarked(markResume bool, nextCurrentProducer string, nextState State, nextError string, nextHistory []HistoryEntry, consumePause, clearConditions bool, routedTo string, transient TransientClass, parentNotice, pauseReason string) error`
- `internal/shedengine#Shed.rerouteStuck`
- `internal/shedengine#Shed.routedState`
- `internal/shedengine#conditionReason`
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#StepResult`
- `internal/shedengine#TestStep_PauseRequested`
- `internal/shedengine#TestStep_PauseBeforeCondition`
- `internal/shedengine#TestStep_PauseAfterCondition`
- `internal/shedengine#TestRun_CancelledBetweenProducers`
- `internal/shedengine#TestRun_CancelledDuringProducerCall`
- `internal/shedengine#TestRun_ResumeAfterPause`
- `internal/shedengine#TestRelaunch`
- `internal/shedengine#TestGoto_MovesBlockedRunToTarget`
- `internal/shedengine#TestStatus_JSONRoundTrip`
- `internal/shedengine/doc.go`

**Intent:** D4 in Shed: every write of `paused` records why.

`Status` gains four Shed-tolerant fields, each `omitempty`: `PauseReason` (`pause_reason`), Shed-owned like `state`; and the caller labels `PauseRequestedBy` (`pause_requested_by`), `PauseBeforeBy` (`pause_before_by`) and `PauseAfterBy` (`pause_after_by`), outside-set beside the request they accompany.
`persistMarked` takes the reason as its new last parameter: a write recording `StatePaused` sets `PauseReason` to it, and every other write clears it; `persistTransient` passes it through, `persist` passes "".
It clears `PauseRequestedBy` in the same write that consumes `pause_requested`, and both `By` labels in the write that clears the conditions.
The reasons, each composed from the status read at step 1 or, for `pause_after`, the fresh read `routedState` already makes:
- a pause request: `pause requested`, followed by ` by <label>` when `PauseRequestedBy` is set;
- a cancelled context, at step 3 or during the call: `cancelled`;
- a fired `pause_before` or `pause_after` condition: `conditionReason`'s text followed by ` by <label>` from the condition's own label when one is set, which `routedState` now returns as its reason.
`StepResult.Reason` on a paused step is the same text the file records, and its doc comment says so.
`Goto` writes `goto to <row>`, followed by ` by <by>` when `GotoRequest.By` is set, and clears the three labels with the conditions it clears.
`Relaunch` with a target it moves the run onto writes `re-launched onto <row>` and keeps the condition labels it keeps; a `Relaunch` with no target, or whose target is already in effect, writes the status unchanged, `pause_reason` included.
Every outside writer (the pause verb, `RecordResumer`) decodes and re-encodes the whole `Status`, so it carries the new fields through.
`doc.go`'s field-ownership section adds `pause_reason` to the Shed-owned set and the labels to the outside-set fields, cleared with their request.

Tests extend the tests of each writer: `TestStep_PauseRequested` (a request with and without a label: `pause requested by orch` and `pause requested`, persisted and returned, and the label cleared), `TestStep_PauseBeforeCondition` and `TestStep_PauseAfterCondition` (the condition reason with and without its label, both labels cleared), `TestRun_CancelledBetweenProducers` and `TestRun_CancelledDuringProducerCall` (`cancelled` persisted by the step-3 arm and by the in-call cancellation arm), `TestRun_ResumeAfterPause` (the resume write clears `pause_reason`), `TestGoto_MovesBlockedRunToTarget` (`goto to <row>`, and with `By: shed` `goto to <row> by shed`, labels cleared), `TestRelaunch` (a moving target writes `re-launched onto A` and keeps `pause_before_by` and `pause_after_by`; the no-target and in-effect rows seed a paused status literal carrying `pause_reason` `pause requested by orch` and both condition labels, and assert the file unchanged, so a write that cleared the reason or a label fails them), and `TestStatus_JSONRoundTrip` (a file without the four keys decodes, and the keys round-trip).

**ImpactSummary:** Re-signs two private persist helpers whose callers all sit in shedengine (persist, stepLocked, rerouteStuck); adds tolerant status fields every reader carries through.

**Commit:** 3: shed: every write of paused records its reason and the requester's label
