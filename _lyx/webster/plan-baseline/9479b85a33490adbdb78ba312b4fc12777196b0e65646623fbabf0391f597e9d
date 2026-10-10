# Card 4 — shedengine-stop-conditions

**Edit:**
- `internal/shedengine#Status`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Shed.persist`
- `internal/shedengine#Shed.persistTransient` -> `func (s *Shed) persistTransient(nextCurrentProducer string, nextState State, nextError string, nextHistory []HistoryEntry, consumePause, clearConditions bool, routedTo string, transient TransientClass, parentNotice string) error`
- `internal/shedengine#Goto`
- `internal/shedengine/doc.go`
- `internal/shedengine/step_test.go`
- `internal/shedengine/status_test.go`
- `internal/shedengine/goto_test.go`
- `contracts/specs/loom-status-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/shedengine#conditionReason`

**Intent:** The engine honours the two stop conditions of D3 wherever it honours `pause_requested`, so the Go driver `lyx loom run` and the loop (card 11) both stop at them through one site.
`Status` gains `PauseBefore` and `PauseAfter` (`pause_before`, `pause_after`, `omitempty`), their own ownership class: an outside actor sets and removes them through `pause` (card 6), and Shed writes them cleared only in the persist that records `paused` for a fired condition and in `Goto`'s persist.
In `stepLocked` step 3, `before` fires when `current_producer` equals its target: the persist writes `paused` at the current row with `clearConditions` true and a `Reason` of `conditionReason("before", target)`, which renders `paused before <target>, as requested`; the `pause_requested` flag is consumed only when it was set, as today.
After a producer returns with a routed running outcome (Done to `OnDone`, a bounced Stuck to `OnStuck`), `after` fires when the producer that ran equals its target: the one persist writes `paused` with `current_producer` at the routed row, both conditions cleared and the reason `paused after <target>, as requested`; a halt outcome (`blocked`, `awaiting`, `done`) wins and leaves the condition recorded, and a producer error leaves it too.
A condition that does not fire is carried through every other write unchanged, so one whose target already ran fires on the next pass that reaches the target.
`persistTransient` takes `clearConditions`; `persist` passes false, and the two condition pauses and the bare pause write call `persistTransient` directly, the bare pause with false, so an operator's bare pause never drops a recorded condition.
`Goto` clears both conditions in its mutate beside `pause_requested`.
`StepResult.Reason` carries the condition reason, so the step envelope's `reason` tells the driver which stop this is.
`step_test.go` gains `TestStep_PauseBeforeCondition` (fires at the target with the reason and clears it, leaves `pause_requested` as it was, does not fire at another row) and `TestStep_PauseAfterCondition`, a table over a Done routed on, a bounced Stuck routed to its `OnStuck`, a halt of the target leaving the condition recorded, and a target that already ran firing on the next pass.
`status_test.go`'s round-trip test gains rows for the two fields and their omission when empty; `goto_test.go`'s move test asserts both are cleared.
`doc.go`'s field-ownership paragraph names the outside-set, both-cleared class; `loom-status-spec.md`'s schema block and per-field notes document the two keys.
`test-budget.yaml` raises `internal/shedengine` from 130 to 134.

**ImpactSummary:** `persistTransient`'s callers are `persist` and `stepLocked`, both in this card; every other package reads `Status` through its JSON shape, which only gains two optional keys.
