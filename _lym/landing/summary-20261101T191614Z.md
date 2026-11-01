# A stopped seat's Wait ends the run instead of blaming reed

## What happened

In the `plan-split-bench` study, the `run-verbs-shared` split arm failed Parts-Plan-Write with `chair wait: reed did not track strand ... on 2 consecutive liveness checks`.
The chair strand had left `lym reed list` while some part writers were still running.

Reed did not lose the strand.
The session transcripts of the scratch worktree give the order of events:

1. The `landing-describe` part writer failed its `plan-part` gate (`part-export-missing`) on each turn end.
   It was waiting for a ruling the chair had asked the parent for, and it used up its gate budget around 17:25:05Z.
2. A writer whose run ends without passing its gate and Accept ends the step.
   `seatengine.join` saw the step ender fire, stopped the notice sender and called `Stop` on the chair.
   `Run.Stop` recorded the stop on the handle and removed the chair's strand through reed.
   This is why the chair vanished from reed while the writers ran.
3. The chair's `Wait` was still polling.
   `checkLivenessTick` found the strand untracked and, since the chair's outputs did not exist, returned `errStrandNotTracked`.
   After two such checks `Wait` failed with that mechanism error instead of finalizing.
4. `join` passed the wait error up as `seatengine: chair wait: ...`.
   `MultiLLMProducer` therefore took its `errorExit` path, not `mapOutcome`'s `writerExit`, and the stop named reed rather than the writer that ended the step.
   `join` then stopped every writer, which is why `lym reed status` later listed no strands.

`Run.Stop`'s contract and the package doc already said a Wait on a stopped handle finalizes with the stop mark.
Only `finalize` honored the mark; the liveness path never reached `finalize` for an untracked strand.

## The change

In `internal/shuttleengine`, `checkLivenessTick` now asks `markedStopped`, a new accessor that reads the handle's stop mark under its record lock, before it returns `errStrandNotTracked`.
A stop-marked handle whose strand reed no longer lists ends the run `OutcomeDied`.
`finalize` then records the run as `stopped`, unless all its output files exist; the existing file-contract check still comes first.
An unmarked handle keeps the old answer, since an untracked strand there is still reed's bookkeeping going away under a live agent.

With this change, the failed step returns no error from the seat run.
`mapOutcome` finds the writer's `EndedStep` and maps it through `writerExit`: Stuck, naming the part and its failed gate.

Docs updated: the doc comments of `Wait` and `checkLivenessTick`, and the stop paragraph of the package doc.
The audited negative-verdict return sites in `completionsignal_enforcement_test.go` now count three `OutcomeDied` returns in `checkLivenessTick`, each behind `allOutputFilesExist`.

## Design decisions

- The fix goes in shuttle's `Wait`, not in reed or in `seatengine.join`.
  The chair was stopped on purpose, so keeping it tracked would be wrong.
  The defect was the stopped handle's `Wait` misreading its own stop.
- Only the untracked answer honors the stop mark.
  `Stop` removes the strand, so a cleared pane binding is not something a stop produces, and that answer is left as it was.

## Tests

`TestStop_WaitThatSeesTheStoppedStrandGoEndsDied` stops a handle and then waits on it while reed lists no strand and the output file is missing.
It pins that `Wait` returns `OutcomeDied` with no error, that the record reads `stopped`, and that only the stop's own removal reached reed.
The existing row "strand absent from reed's table is a mechanism failure, not died" still covers the unmarked handle.

## Follow-up for the parent

The step ended because a writer waiting for the chair's ruling spent its gate budget on turn ends where it changed nothing.
A writer that waits on its chair can therefore end the whole Parts-Plan-Write step.
This is worth a board note for the `plan-split-tune` work.
