```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run lens fix-code-correctness over the F1–F4 code changes in `wait.go` and `run.go` and the tests in `waiting_test.go`.

I found no BLOCKING, MEDIUM or LOW defects. I traced each changed branch by hand from the changed files in context.

- **Stale-wait clear in `pollEventsTick`.**
  - Input: gated run, files present, one shell. Tick 1 reads `WAIT`. A later tick reads `START` after the bound has passed.
  - `START`'s end offset (27) is past `waitingTurnEnd.offset` (21), so `waitingTasks` and `waitingTurnEnd` are cleared and the tasks are kept in `countedTasks`. `expiredTurnEnd` then returns "" and the gated Done no longer fires mid-turn.
  - After an ungated expiry into a held turn end, `waitingTasks` is already nil. The `len > 0` guard stops `countedTasks` being overwritten with nil.
  - A following `STOP` sets `countedTasks = nil` at the events branch, so `EndedShells` ends up empty.
- **Gated same-batch branch.**
  - Input: `WAIT` and `START` in one batch, files present, only unawaited shells.
  - The branch moves the tasks to `countedTasks`, clears the wait and returns "" without queueing a nudge. With the branch removed, the nudge would be sent.
- **`queueShellNudge` and `sendShellNudge`.**
  - A queue followed by a successful send marks the ids named.
  - A pause drop or a failed `sendWithin` leaves them unmarked.
  - `sendShellNudge` runs every loop iteration, including the events-error path, and clears the queue each time, so no stale `shellNudgeIDs` survive.
  - The new `shellNudgeIDs` field is read only here.
- **Tests fail with the defect in place.**
  - `TestPollEventsTick_TurnStartEndsTheGatedShellWait`: the pre-fix `expiredTurnEnd` returns Done at the bound when the `START` tick has no events, so the test fails.
  - Row "turn start past the waiting turn end": removing the early return makes the run send once, against `wantSent` 0.
  - Row "pause covering the run": marking before the send would leave 0 sends, against `wantSent` 1.
  - Row "failed send": marking before the send would leave 1 send, against `wantSent` 2.
  - The gate run confirmed the new tests pass, including the failed-send row. Its `SendText` error is not retried in place.
  - Renaming `TestWait_GatedShellNudgeNamesEachShellOnce` to `TestWait_GatedShellNudge` leaves no other reference, as the earlier grep showed.

Outside the fix's scope: for an ungated run with an output file missing, a `WAIT` and a later `START` in the same batch can still expire into a held turn end when the shells are already past the bound. Only the gated same-batch branch got the new guard.

Further tool calls: none, because the context already held every changed file and the gate output.
