# Card 3 — shed-cancelled-state

**Create:**
- `plan:internal/shedengine#StateCancelled` -> `const StateCancelled State = "cancelled"`
- `plan:internal/shedengine#CancelIntent` -> `type CancelIntent struct`
- `plan:internal/shedengine#CancelFinish` -> `type CancelFinish struct`
- `plan:internal/shedengine#PauseIntent` -> `type PauseIntent struct`
- `plan:internal/shedengine#TestStep_CancelledRunIsNeverStepped` -> `func TestStep_CancelledRunIsNeverStepped(t *testing.T)`
- `plan:internal/shedengine#TestStep_CarriesTheRunIntents` -> `func TestStep_CarriesTheRunIntents(t *testing.T)`

**Edit:**
- `internal/shedengine#Status`
- `internal/shedengine#State.valid`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#TestState_Valid`
- `internal/shedengine#TestStatus_JSONRoundTrip`

**Uses:**
- `internal/shedengine/doc.go`

**ImpactSummary:** the status file gains two optional keys and a seventh state; every existing record decodes and steps as before.

**Intent:**
Give the batten record its tracked cancel and pause intents and the shed state `cancelled` (D3).

`CancelIntent` holds `StartedAt` (RFC 3339), `Machine`, `Remove` and `Finish *CancelFinish`.
`CancelFinish` holds `At` (RFC 3339) and `Remove`; a finish with `Remove` false is a kept pair.
`PauseIntent` holds `By` (the caller label) and `At` (RFC 3339).
`Status` gains `Cancel *CancelIntent` with the JSON key `cancel` and `Pause *PauseIntent` with the key `pause`, both `omitempty`.
They are external-writer-owned: an outside writer sets them and Shed carries them through every persist,
which `persistMarked`'s read-modify-write already does; the field-ownership comment on `Status` names them.

`StateCancelled` is the state a finished cancel writes, never `done`, so nothing that takes a done run takes a cancelled one.
`State.valid` admits it, and the "six legal values" comment becomes seven.
`Shed.stepLocked` refuses a cancelled status before its lookup, as a finished run that Shed never steps:
the error names the status file and, as way forward, seeding a new run.
`Goto` and `Relaunch` refuse it beside `done`, worded the same way.

`TestStep_CancelledRunIsNeverStepped` reaches each of the three refusals over a status file whose state is cancelled and asserts the file is unchanged.
`TestStep_CarriesTheRunIntents` writes both intents beside a running status, steps it once, and asserts the persisted file still carries them.
`TestState_Valid` gains the cancelled row and `TestStatus_JSONRoundTrip` round-trips both intents and their absence.
All of them are tier 0.

**Commit:** 3: shedengine: a cancelled state and the run's cancel and pause intents in the status file
