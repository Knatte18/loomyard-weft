# Card 2 — shed-cancelled-state-intents

**Create:**
- `plan:internal/shedengine#StateCancelled` -> `const StateCancelled State = "cancelled"`
- `plan:internal/shedengine#CancelIntent` -> `type CancelIntent struct`
- `plan:internal/shedengine#CancelFinish` -> `type CancelFinish struct`
- `plan:internal/shedengine#PauseIntent` -> `type PauseIntent struct`
- `plan:internal/shedengine#TestCancelledRun_NeverStepsGoesToOrRelaunches` -> `func TestCancelledRun_NeverStepsGoesToOrRelaunches(t *testing.T)`

**Edit:**
- `internal/shedengine#State.valid`
- `internal/shedengine#Status`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#TestState_Valid`
- `internal/shedengine#TestStatus_JSONRoundTrip`
- `internal/shedengine/doc.go`

**Intent:** Give the batten record, a shed status file, the fields D3 moves into tracked records, and a terminal state for a finished cancel.

- `StateCancelled` joins the legal states; `State.valid` accepts it.
  Fix the "six legal State values" comment by naming the set without a count.
- `CancelIntent` holds `StartedAt` (RFC 3339), `Machine` (the machine that started the cancel), `Remove` (remove or keep the pair) and `Finish *CancelFinish`.
- `CancelFinish` holds `EndedAt` (RFC 3339) and `Remove`, the way the cancel ended.
- `PauseIntent` holds `By` (the caller label of the pause), `Machine` and `PausedAt` (RFC 3339).
- `Status` gains `CancelIntent *CancelIntent` (`json:"cancel_intent,omitempty"`) and `PauseIntent *PauseIntent` (`json:"pause_intent,omitempty"`).
  Both are outside-writer-owned, like `Product`: Shed never sets or clears them.
  Every persist reads the file through `internal/state` and rewrites only its own fields, so an intent written between two persists survives.
  Say so in the `Status` field-ownership comment.
- A cancelled status is terminal like a done one, but never reads as done:
  `Shed.stepLocked` returns an error for it naming the cancelled run and the way forward, "seed a new run";
  `Goto` and `Relaunch` refuse it the way they refuse a done status.
- `doc.go`'s persisted state vocabulary adds `cancelled`, set only by an outside writer, and names the two intent fields.

Tests, all tier0 in existing test files:
`TestState_Valid` gains a `cancelled` row and an `awaiting` row;
`TestStatus_JSONRoundTrip` round-trips both intents, the finish included, and decodes a status without them as nil;
`TestCancelledRun_NeverStepsGoesToOrRelaunches` seeds a cancelled status and asserts that a step, a goto and a relaunch each refuse with the file unchanged.

**ImpactSummary:** Adds one state and two optional status fields; shed's persists carry them through, and the new state is refused by step, goto and relaunch only.

**Commit:** 2: shed-cancelled-state-intents
