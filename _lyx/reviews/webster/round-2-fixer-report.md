# Fixer report — Webster round 2

## Fixed

- **F1** (MEDIUM, decision) — `cd6235aa8`.
  The shed section of `contracts/specs/refusal-spec.md` gains two rows.
  `seat strand name held` (correctness halt) covers `checkName` on a `MultiLLM` row's chair; its way forward quotes `lyx reed remove --name <name>`, then re-step the row, and the trigger notes that an advisor in the same case is recorded as not started.
  `seat not stopped` (transient, matching the existing `producer re-run: superseded strand not removed` row) covers `stopSeat`'s `ErrSeatNotStopped`; its way forward quotes `lyx reed remove <guid>`, then re-step the row.
  The decision: the seat engine's refusals surface only through a shed producer row, and shed is a bound module, so they take rows rather than an out-of-scope entry.
  Tests already reach both: the chair and advisor rows of `TestEngine_Run_StartFailureEndingTheRunStopsTheStartedAdvisors` and the formed-name-mismatch cases in `engine_test.go`.
- **F2** (LOW, consistency) — `1f5a8ef7e`.
  `internal/shedadapters/doc.go` now says an `ErrSeatNotStopped` error keeps its way forward even under cancellation, and that a probe's halts `Call` before the archive while a run's halts it after.
  This matches `Call`'s order and the `wantArchived` values of `TestMultiLLMProducer_ErrorExits`.

`go test` over `contracts/...`, `cmd/lyx/...`, `internal/shedadapters/...` and `internal/seatengine/...` passes, and `lyx loom lint-comments` reports ok.

## Disputed

None.

## Deferred

None.
