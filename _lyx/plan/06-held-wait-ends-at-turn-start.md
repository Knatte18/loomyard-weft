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
- `internal/shuttleengine#turnStartOffsets`
- `internal/shuttleengine#Run.endHeldWait`
- `internal/shuttleengine#Run.loopClassification`

**Intent:** D5 of the decision record: the held wait ends at the first turn start read past the held turn end's offset, so a parent's answer that starts the agent working returns the loop to `running` while the session reads `busy`, and the disagreement line says which side stands.

`waitState` gains `heldOffset`, the events-file offset just past the held turn end on show, zero while the held wait is off; `beginHeldWait` takes that offset and records it beside `heldStart`, keeping an existing hold's start and offset.
`Wait` passes `held.offset` at its one call.
`Run` gains `turnStartRead`, true when the latest `pollEventsTick` read a turn start positioned past the latest turn end the loop holds.
That reference is one offset, never two: the offset of the held turn end the tick itself returns when it returns one, else `heldOffset` while a held wait is on show.
So a batch holding a turn start and then a new held turn end, read while an earlier hold is on show, leaves `turnStartRead` false: the turn start lies before the new turn end, the old hold ends through `eventsRead`, and the new hold shows.
`pollEventsTick` resets it with `eventsRead` at its start and sets it from the batch's `turnStartOffsets`, card 5's read right after `ParseEvents` and before the `len(events) == 0` return, so a turn start read alone in its tick, which yields no event, still ends the hold; it is never set from the shadow's fold.
`Wait` ends the held wait when either `eventsRead` or `turnStartRead` is set, which makes `loopClassification` read `running` on that tick and the wait mark clear.
When a tick returns a held turn end and `turnStartRead` is set, the same batch holds the next turn's start past it: `Wait` logs the held turn end and sends the hold notice as today, and does not call `beginHeldWait`, so the loop never shows `held` across a turn already under way.
Nothing else moves: the hold notice went at the turn end, `NotifiedOffset`, the deadline and the liveness check are untouched, and the held marker's readers outside shuttle treat a cleared marker as they treat an absent one.

The `shuttle: session state disagrees` Warn in `logSessionState` gains a fixed pair `stands=loop` at its end, saying the loop's classification is the one every decision follows; no action follows a disagreement, as before.
The package doc's session-state paragraph says the session state is a shadow reading the loop never acts on and names the field; its wait-mark paragraph says the held wait ends at the next turn start, turn end or ask, which was already its claim and is now true.

Tests.
`TestRun_Wait_HeldTurnEnd` gains three rows with `sessionFakeEngine`: a `STOP:question` holds, a `START` appended alone clears the marker at the next step with no new hold, and output plus `STOP:done` finish (AC13); a `STOP:question` and a `START` appended in one step leave the marker not held at the next step, and output plus `STOP:done` finish (AC13, one batch); a `STOP:question` holds, then a `START` and a second `STOP:question` appended in one step leave the marker held, and output plus `STOP:done` finish (the reference is the tick's own held turn end).
The hold notice for that one-batch shape is pinned by card 5's zero-`PromptOffset` row of `TestWait_HeldTurnEndNotice`, which this card leaves passing.
`TestWait_LogsSessionStateBesideItsClassification` gains a row in which a held turn end is followed by a `START`, asserting the logged change `busy/turn running running` and no disagreement, and its existing disagreement row asserts the Warn line carries `stands=loop` (AC13, AC14).
No new top-level test is added, so the package's row in `cmd/lyx/testdata/test-budget.yaml` stays.

**ImpactSummary:** One re-signed method with its one caller in `Wait`, two fields, two conditions in `Wait`, one log field; display and classification only, no verdict, notice or return changes, and nothing outside `internal/shuttleengine` changes.
