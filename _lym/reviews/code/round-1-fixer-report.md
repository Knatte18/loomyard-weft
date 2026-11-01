# Fixer-report, round 1

## Changes

- F1 (MEDIUM), fixed as proposed, commit `58a00783`.
  Added the stop-marked exception to the `OutcomeDied` doc in `internal/shuttleengine/engine.go`, the file header of `internal/shuttleengine/wait.go`, and `docs/overview.md` (shuttle bullet).
  Each states that a handle whose own `Stop` removed the strand ends `OutcomeDied`, recorded as stopped unless the files show the run done, matching `Wait`'s doc.
- F2 (LOW), fixed as proposed, commit `445c693c`.
  `TestStop_WaitThatSeesTheStoppedStrandGoEndsDied` in `internal/shuttleengine/stop_test.go` is table-driven with rows `OutputMissing` (expects `OutcomeDied`, recorded stopped) and `OutputPresent` (output written after `Stop`, expects `OutcomeDone`, nil error).
  No separate test function added.

`lym gate test ./internal/shuttleengine` passed at tier0 (1201 targets, 0 failed).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
