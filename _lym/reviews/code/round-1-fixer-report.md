# Round 1 fixer-report

## Changes

- **F1 (MEDIUM)**, fixed as proposed: `internal/seatengine/resume_test.go` gains two subtests in `TestEngine_Resume_Writers`.
  - A writer not found at resume whose output is missing: asserts `EndedStep`, empty `StrandGUID` and `Outcome`, `Accept` never called, the chair stopped once and no line sent.
  - A writer not found whose output exists and whose `Accept` errors: asserts `EndedStep`, `AcceptErr` set, the chair stopped once and no line sent.
  - No production code changed.
  - Commit `seatengine: fix F1 — pin the lost-writer branch that ends the step at resume`.
- Self-check follow-up: both subtests now run `Resume` in a goroutine read through `await`, so a regression fails by assertion instead of hanging, and assert `Notified` false, which is deterministic where `len(chair.sent)` is not.
  - Commit `seatengine: fix self-check findings — fail the lost-writer subtests by assertion, not hang`.

`lym gate test ./internal/seatengine` passes after each commit.

## Self-check

- fix-code-correctness, LOW, subtests hang instead of failing with the defect in place: fixed (goroutine and `await`).
- fix-proposal, LOW, the no-line check passes when a notice is queued but undelivered: fixed differently, by adding a `Notified` false assertion on the writer result; the `len(chair.sent)` check stays.
- fix-proposal, LOW, the Accept-fails subtest does not assert `Accept` ran: disputed, premise false; the lens itself notes that a set `AcceptErr` proves the call.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

- fix-proposal LOW, `Accept` not asserted in the Accept-fails subtest. Ground: premise false. Evidence: `errors.Is(got.AcceptErr, rejected)` can only hold if `seatRun.accept` called the hook (`internal/seatengine/engine.go:157-166`).

## Deferred

None.
