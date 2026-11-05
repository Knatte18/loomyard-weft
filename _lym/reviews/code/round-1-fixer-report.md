# Fixer-report, code review round 1

## Changes

Both findings fixed as proposed, in `internal/creelengine/deliver_test.go`, in one commit (`99d8707a`) because both edit the one test function's neighbourhood.

- **F1:** added the row `MailAfterAnAcknowledgedRingGetsNoBellBeforeTheWindowWhenTheReadingHasNoTurnEndTime` to `TestScheduler_HookWake`, after the two new rows.
  It sets `SetTurn(alpha, false, time.Time{})`, runs `ackedRing` for "one", posts "two" under `settle`, and asserts `untilBell` returns at least 30 seconds.
- **F2:** extended the `TestScheduler_HookWake` doc comment with a line saying the bell is also held within 30 seconds of a ring while no turn end was read after it, since the woken turn's Stop hands out the mail.

`lym gate test ./internal/creelengine` passed: tier0 341 targets, 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

No fork returned without a report.

## Disputed

None.

## Deferred

None.
