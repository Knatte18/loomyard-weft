## Fixer-report, round 1

All six findings are fixed.
The tier0 gate for `./internal/shuttleengine`, `./internal/shuttleengine/claudeengine` and `./internal/fabricengine` passed after the first round of edits, and for the first two packages again after the self-check edits.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  `DeliverPendingPrompt` (`internal/shuttleengine/run.go`) calls `clearPendingSend` before `sendThroughPause` and logs a failure at Warn, so `retryLanded` finds no record and the line is typed.
  `promptDelivered` is set only after the send.
  Rows added: `TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted` (pending-send record plus an operator turn start) and `TestWait_HeldTurnEndNotice` (same record at a held turn end).
  The "Pending prompt" section of `doc.go` now says the record is cleared first.
- **F2 (MEDIUM), fixed as proposed, with two departures.**
  `TestWait_HeldTurnEndNotice` gets rows for a run covered by a pause (nothing typed), for a turn start read after the held turn end in the same batch (nothing typed), and for a failed prompt send.
  `TestAttach_DeliversTheDeferredPrompt` gets a failed-send row that asserts the send was attempted and that Attach still returns the run.
  Departure 1: the failed-send row uses a send that fails once (`echoReed.sendFailures`) over a turn with no instruction, so the first held turn end falls through to the hold notice and the second types the prompt.
  A permanent failure cannot show the retry, and an operator-typed turn would send no notice, so the review's "hold notice still sent" does not hold for it.
  Departure 2: no new test function, as the review asked.
  I confirmed the `turnStartRead` row fails when the guard is removed (mutation run, guard restored afterwards).
- **F3 (LOW), fixed with my own change.**
  Proposed: return early in `deliverPromptOnAttach` when the run is covered and "leave the attach delivery to the first held turn end".
  Not made as written: two self-check forks showed that an attached run whose driver died before the prompt went out produces no turn end, so a covered attach would lose its prompt for good.
  Made instead: `deliverPromptOnAttach` runs at the top of each Wait tick after `observePause`, judges the delivery once per handle (`Run.attachPromptTried`) at the first tick no pause covers, and does nothing under a pause.
  Wait keeps its liveness and deadline polling during the pause, and the prompt is typed after the lift.
  The attach row asserts the line is typed after the pause lifts.
- **F4 (LOW), fixed as proposed.**
  `writeOpenFiles` (`claudeengine/command.go`) removes the directory it created on every later error return.
  Rows added: `TestOpenLaunch` (append-prompt write fails) and `TestRunner_Open` (`AddStrand` fails after the engine created the session directory), which also covers the existing cleanup in `open.go`.
  `OpenLaunch`'s doc comment names the cleanup.
- **F5 (LOW), fixed as proposed.**
  The hunks of `internal/fabricengine/recordswritelock_test.go` are reverted.
  The flaky wait is not fixed here; it is left for the run's parent to land on its own.
- **F6 (NIT), fixed as proposed.**
  The operator-turn and own-turn-start rows are dropped from `TestAttach_DeliversTheDeferredPrompt`.

Commits, in order: F5, F4, F1+F2+F3+F6 together (one commit, since the hunks share files), then the F3 change above.
The F1, F2, F3 and F6 commit message names all four ids; one commit per finding was not possible with the shared files and no interactive staging.

## Self-check

- fix-code-correctness, MEDIUM: covered attach loses its prompt for good. Fixed with my own change (see F3).
- fix-proposal, MEDIUM: same finding as above, merged. Fixed with my own change (see F3).
- fix-code-correctness, LOW: the failed-send attach row cannot detect a missing send attempt. Fixed: the row asserts `SendTextCalls` is non-empty.
- fix-code-correctness, LOW: the `turnStartRead` row might not exercise the guard. Fixed by verification: with the guard removed the row fails, and the guard is restored.
- fix-consistency, LOW: `doc.go` "Pending prompt" section misses the covering-pause condition on entry delivery and the pending-send clear. Fixed as proposed.
- fix-consistency, LOW: `OpenLaunch` doc does not name the directory cleanup. Fixed as proposed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
