# Fixer-report, darn round 1

## Changes

All four findings fixed as proposed; commit `ae2831c1` holds F1–F4 together, since F2–F4's tests share one file with F1's regression test.

- F1: `withdrawQueuedCompact` (`internal/orchengine/watcher.go`) sets `State.ResumeLineWithdrawn` from the withdrawn list and saves explicitly when the post-withdraw `InjectOutcome` fails, before `holdOnCreelFailure`.
  The doc comment names the kept line.
- F2: subtest `AFailedReadAfterTheWithdrawKeepsTheLineAndTheNextTickClearsIt` in `TestCompact_BoundaryWithdrawsAStillQueuedCompact`.
  A settle tick drains the start color's outcome read first, so the scripted `nil` then `errBoom` hit the compact's pre-read and post-withdraw read.
- F3: row `Unconfirmed_CompletesWithNoFurtherWithdraw` in the delivery-attempt table.
- F4: subtest `AWithdrawnResumeLineHasItsMarkClearedWhileTheCompactIsStillInFlight`, a separate subtest because the table has no field for a withdrawn list.

Self-check follow-up, commit after `ae2831c1`: the `ResumeLineWithdrawn` field comment in `state.go` and a compact-boundary entry in `doc.go`.

`lyx gate test ./internal/orchengine` passes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `state.go` field comment names only the fresh-launch retry as setter. Fixed as proposed.
- fix-consistency, LOW: `doc.go` has no entry for the compact-boundary withdraw. Fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
