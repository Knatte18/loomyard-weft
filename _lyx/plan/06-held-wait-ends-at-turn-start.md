# Card 6 — held-wait-ends-at-turn-start

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.pollEventsTick`
- `internal/shuttleengine#waitState`
- `internal/shuttleengine#Run.beginHeldWait` -> `func (run *Run) beginHeldWait(offset int64)`
- `internal/shuttleengine#Run.logSessionState`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#TestRun_Wait_HeldTurnEnd`
- `internal/shuttleengine#TestWait_LogsSessionStateBesideItsClassification`

**Uses:**
- `plan:internal/shuttleengine#turnStartOffsets`
- `internal/shuttleengine#Run.endHeldWait`
- `internal/shuttleengine#Run.loopClassification`

**Intent:** D5 of the decision record: the held wait ends at the first turn start read past the held turn end's offset, so a parent's answer that starts the agent working returns the loop to `running` while the session reads `busy`, and the disagreement line says which side stands.

`waitState` gains `heldOffset`, the events-file offset just past the held turn end on show, zero while the held wait is off; `beginHeldWait` takes that offset and records it beside `heldStart`, keeping an existing hold's start and offset.
`Wait` passes `held.offset` at its one call.
`Run` gains `turnStartRead`, true when the latest `pollEventsTick` read a turn start positioned past `heldOffset` while a held wait was on show; `pollEventsTick` resets it with `eventsRead` at its start and sets it from the batch's `turnStartOffsets`, the same read card 5 makes, so a turn start that yields no event still ends the hold, and never from the shadow's fold.
`Wait` ends the held wait when either `eventsRead` or `turnStartRead` is set, which makes `loopClassification` read `running` on that tick and the wait mark clear.
Nothing else moves: the hold notice went at the turn end, `NotifiedOffset`, the deadline and the liveness check are untouched, and the held marker's readers outside shuttle treat a cleared marker as they treat an absent one.

The `shuttle: session state disagrees` Warn in `logSessionState` gains a fixed pair `stands=loop` at its end, saying the loop's classification is the one every decision follows; no action follows a disagreement, as before.
The package doc's session-state paragraph says the session state is a shadow reading the loop never acts on and names the field; its wait-mark paragraph says the held wait ends at the next turn start, turn end or ask, which was already its claim and is now true.

Tests.
`TestRun_Wait_HeldTurnEnd` gains a row with `sessionFakeEngine`: a `STOP:question` holds, a `START` appended alone clears the marker at the next step with no new hold, and output plus `STOP:done` finish (AC13).
`TestWait_LogsSessionStateBesideItsClassification` gains a row in which a held turn end is followed by a `START`, asserting the logged change `busy/turn running running` and no disagreement, and its existing disagreement row asserts the Warn line carries `stands=loop` (AC13, AC14).
No new top-level test is added, so the package's row in `cmd/lyx/testdata/test-budget.yaml` stays.

**ImpactSummary:** One re-signed method with its one caller in `Wait`, two fields, one condition in `Wait`, one log field; display and classification only, no verdict, notice or return changes, and nothing outside `internal/shuttleengine` changes.
