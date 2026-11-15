# Card 1 — shed-run-intents

**Create:**
- `plan:internal/shedengine#StateCancelled` -> `const StateCancelled State = "cancelled"`
- `plan:internal/shedengine#CancelIntent` -> `type CancelIntent struct`
- `plan:internal/shedengine#CancelFinish` -> `type CancelFinish struct`
- `plan:internal/shedengine#PauseIntent` -> `type PauseIntent struct`

**Edit:**
- `internal/shedengine#Status`
- `internal/shedengine#State.valid`
- `internal/shedengine#TestState_Valid`
- `internal/shedengine#TestStatus_JSONRoundTrip`
- `internal/shedengine#TestRun_ExternalWriteSurvivesPersist`

**Uses:**
- `internal/shedengine/doc.go`

**Intent:** Give the batten run record, `shedengine.Status`, the two run decisions D3 moves out of `.lym`.

- `CancelIntent` holds `StartedAt` (RFC 3339), `Machine` (the machine that started the cancel), `Remove` (true for `--remove`, false for keep) and `Finish *CancelFinish`.
  `CancelFinish` holds `EndedAt` (RFC 3339) and `Remove`, whether the finished cancel removed the pair or kept it.
- `PauseIntent` holds `By` (the caller label of the pause) and `At` (RFC 3339).
- `Status` gains `Cancel *CancelIntent` with JSON key `cancel` and `PauseIntent *PauseIntent` with JSON key `pause_intent`, both `omitempty`.
  Their doc comments say they are outside-owned, as `PauseRequested` is: Shed never writes them.
  A status file written before the fields existed decodes them as nil.
- `StateCancelled` is the state a finished cancel writes, never `done`, so the dobby's done-run clear never takes a cancelled run.
  `valid` admits it, so a step or goto read of a cancelled record does not fail as an invalid state; the act rule (card 3) refuses driving it.
  The doc comment above the state constants drops "six" for the closed set's own list.

Tests, all tier0:
- `TestState_Valid` gains the row `cancelled` valid.
- `TestStatus_JSONRoundTrip` gains a status with a finished cancel intent and a pause intent that round-trips, and an old file with neither key that decodes both as nil.
- `TestRun_ExternalWriteSurvivesPersist` gains the row that a `Cancel` and a `PauseIntent` written by an outside writer between two persists survive them,
  which pins that Shed's read-modify-write carries both fields through.

**ImpactSummary:** Adds two optional fields and one state to the status record; no existing caller changes, and readers that switch on the state meet `cancelled` only once card 7 writes it.
