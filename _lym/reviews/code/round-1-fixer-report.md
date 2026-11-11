# Fixer-report, code review round 1

F2 to F9 are fixed in two commits (`327ed70ad`, `20d2fdcd7`), not one commit per finding.
F1 is `decision`-class and went to the parent; its ruling is under Rulings.
Tier 0 and tier 1 of `lym gate test` pass over `./internal/shedverbs`.

## Changes

- **F2** — `loop_unix_integration_test.go`: new `TestLoopIntegration_KeeperWaitsThroughATerminatingSignal`.
  It sends SIGTERM through an unbuffered `Signals` fake while the loop script still runs.
  The send completes only if the keeper's own goroutine receives it.
  It asserts `keepLoop` has not returned, then returns 1 and writes the exit mark once the script ends.
  A shared `openLoopLog` helper now serves it and the `keep` rows.
- **F3** — `loop_arming_integration_test.go`: two rows in `TestLoopIntegration_ArmingRefusalStopsAsBootstrap`, `shed step --loop-keeper=…` and `loom step --loop-keeper=…`.
  Each runs the built binary and reads the refusal from the loop envelope file, since the keeper prints nothing.
  Fixed differently: the review proposed the keeper flag in place of the detached flag in the existing in-process row.
  That row cannot reach the arming path, because an in-process keeper starts a real loop process instead of refusing.
  The two real-binary rows also cover the second `DetachedLoopID` caller, in `loomcli`.
- **F4** — `loopwait.go`: `deadLoopStop` calls the new `awaitLoopProcessExitMark` before it reads the loop log.
  The wait is at most `keeperMarkLooks` looks, `keeperMarkPoll` apart, through `LoopSpec.Sleep` where set.
  It ends once a `process N exited` mark of the loop is in the log.
  `loopExitEvidence` is split into `loopLogLinesOf` and `loopProcessExitRecorded`.
  New subtest in `TestUntilStop_DeadLoopStopsLoopExited`: the exit mark lands during the wait and reaches `loop.detail`.
  `doc.go` and the `deadLoopStop` doc say so; the `loopExitedEarlyStop` doc says why its waiter needs no wait.
- **F5** — `LoopSpec.Spawn` comment in `loop.go` names the keeper.
- **F6** — `LoopSpec.OnStart` comment in `loop.go` and the sentence in `doc.go` say the keeper runs it whether or not the loop then refuses.
- **F7** — `TestLoopIntegration_ProcessExitsAreMarked`: `keep` returns and asserts `keepLoop`'s code and the `OnStart` count.
  New rows: a clean exit (code 0) and a start failure (could-not-start mark, no `OnStart`, code 1).
- **F8** — `loopwait_test.go`: every row of the in-flight table asserts `loop.status_moved`.
  New rows: a history-only move, and a status file whose `current_producer` is empty.
  The empty row checks the stop's kind and exit code only, because `assertLoopExited` builds its expected text from the status file's producer.
- **F9** — `deadLoopStop` names the status file's producer only when it differs from the in-flight producer.
  `doc.go` is updated and the history-only row pins it.

## Gate repair

The gate refused the round's comments for lines of 120 characters or more.
The comments in `doc.go`, `loop.go`, `loopwait.go`, `loop_arming_integration_test.go` and `loop_unix_integration_test.go` that the gate named are broken at sentence boundaries, one sentence per line, with no change to their meaning.
A second gate pass named one line left in `awaitLoopProcessExitMark`'s comment, now split in two sentences.
The `doc.go` sentence about a loop that refuses before its first step is now two sentences.

A later `lym verify` run failed `TestDaemonWakeups_UnboundedWaitsStayAboveTheFloor` on `keeperMarkPoll`, a sub-second wait.
It is a bounded re-read of at most `keeperMarkLooks` looks, so it carries the `//lym:one-shot` annotation the pattern asks for.
The same run's `internal/reedengine` failure, `TestStrandOps_RealTmux/ResumeDropsAFinishedStrandAndRelaunchesAnUnfinishedOne`, is in a package this round does not touch.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-consistency, LOW, `doc.go` dead-loop description omits the bounded wait: fixed as proposed.
- fix-consistency, LOW, `loop_unix_integration_test.go` file header out of date: fixed as proposed.
- fix-consistency, LOW, `loopExitedEarlyStop` has no wait and does not say why: fixed as proposed, by a comment line.
- fix-affected-set, LOW, `doc.go` opening line says OnStart runs once in the loop: fixed.
- fix-affected-set, LOW, `runLoop` doc restates OnStart without the keeper: fixed.
- fix-affected-set, LOW, `LoopSpec.Executable` and `ArmStop.LoopID` comments name only the detached loop: fixed.
- fix-affected-set, LOW, keeper arming row covered only `shed step`: fixed, with a `loom step --loop-keeper` row.

## Disputed

None.

## Deferred

None.

## Rulings

- **F1** — creel #128 from `ly:LOOM`: the parent recorded that this run leaves out the bounded load-turn retry and the idle time in `lym loom status`, as board task `load-turn-retry-cap`.
  The ruling asks for no code change, and the code already holds the omission.
  Nothing was changed and the parent was not told.
