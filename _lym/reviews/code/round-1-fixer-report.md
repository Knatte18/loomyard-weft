# Fixer-report, round 1

F1 is a `decision`-class finding held by the parent; it is not reported here beyond the ruling below.
F2 to F9 are fixed.
`lym gate test ./internal/shedverbs` (tier 0) and `lym gate test --tier 1 ./internal/shedverbs` pass after the last change.

## Changes

Commits are grouped by file, so a commit carries every finding that touches its files.

- F2 (`loop_unix_integration_test.go`): `TestLoopIntegration_KeeperKeepsWaitingThroughTerminatingSignals` runs `keepLoop` with a `Signals` fake and a loop script that waits on a release file.
  It sends SIGTERM through the fake, asserts the keeper took it and did not return while the loop still ran, then releases the loop and asserts exit code 0 and the loop's exit mark.
  Fixed as proposed.
- F7 (`loop_unix_integration_test.go`): `TestLoopIntegration_KeeperMarksTheLoopProcessExit` is a table of keeper rows that asserts `keepLoop`'s return code and the `OnStart` count for each.
  It gains a `StartFailure` row (missing executable: `could not start` mark, `OnStart` not run, code 1) and a `CleanExit` row (code 0).
  The spawn row moved to its own test, `TestLoopIntegration_SpawnMarksKeeperExit`, because `spawnLoop` returns no exit code.
  Fixed as proposed.
- F3 (`loop_arming_integration_test.go`): a subtest of `TestLoopIntegration_ArmingRefusalStopsAsBootstrap` runs the built binary with `--loop-keeper=loop-int-2`.
  Fixed with my own change.
  The review proposed a table row asserting the refusal on stdout, but a keeper with a loop id writes the refusal to the loop envelope file and prints nothing (`ReportLoopArmError`), so a row that decodes stdout cannot run.
  The subtest asserts empty stdout, then the envelope file under `loop-int-2` holding the bootstrap envelope, and the failed status.
  Reverting `DetachedLoopID` to the detached flag alone leaves the loop id empty, which prints the refusal and writes no file, so the subtest fails.
- F4 (`loopwait.go`): `deadLoopStop` takes `awaitExitMark`.
  Set for a pid record found at start and for a dead end reached with no spawn channel, it makes `awaitLoopProcessMark` poll the loop log up to `keeperMarkPolls` times, `keeperMarkPoll` apart, through `pauseWait`, so `LoopSpec.Sleep` keeps it injectable.
  The teardown and a waiter that spawned the keeper skip it.
  A new subtest of `TestUntilStop_DeadLoopStopsLoopExited` delivers the mark during the second wait.
  Fixed as proposed.
- F5 (`loop.go`): the `LoopSpec.Spawn` comment now says it starts the loop's keeper and returns a channel closed when the keeper exits.
  Fixed as proposed.
- F6 (`loop.go`, `doc.go`): the `OnStart` comment and the `doc.go` sentence state that the keeper runs `OnStart` after starting the loop process whether or not that loop then refuses, and that only the loop's own run skips it on a refusal.
  Fixed as proposed.
- F8 (`loopwait_test.go`): the dead-loop table asserts `loop.status_moved` for every row.
  It gains a `statusProducer` field and two rows, a moved status file with the same producer and one with an empty producer, both wanting the cause without the now-reads clause.
  `assertLoopExited` is skipped for the empty-producer row, because it builds its expected text from the status file's producer.
  Fixed as proposed.
- F9 (`loopwait.go`, `doc.go`): the now-reads clause fires only when the status file's producer differs from the in-flight one.
  Fixed as proposed.

## Gate repair

The comment gate refused nine lines of the round's comments in `doc.go`, `loop.go` and `loopwait.go`: eight ran to 120 characters or more, and one wrapped mid-sentence.
Each is now split at a sentence boundary, one sentence per line, and the `runUntilStop` list item that wrapped reads as two list items.
The wording is unchanged.

A later verify run failed `TestDaemonWakeups_UnboundedWaitsStayAboveTheFloor`: `keeperMarkPoll` (F4) is a 100ms wait, below the one-second floor.
The wait is a bounded re-read of at most `keeperMarkPolls` attempts, so the constant carries the `//lym:one-shot` annotation that the other bounded polls in the repository use.

## Self-check

- fix-consistency, LOW: the waiter docs and `doc.go` did not mention the exit-mark wait. Fixed: `runUntilStop` and `doc.go` now state it.
- fix-affected-set, LOW: `loopwait.go:1` and the `runUntilStop` doc still said the waiter starts the detached loop. Fixed: both now name the keeper.
- fix-code-correctness, LOW: the keeper-signal test left its loop script polling forever on failure. Fixed: a `t.Cleanup` writes the release file.
- fix-proposal, LOW: the give-up path of the exit-mark wait is unpinned. Disputed, premise false: the existing "not recorded" rows of the dead-loop tests run through the full wait and assert the stop is still reported, and the poll count is not a contract.
- fix-proposal, LOW: the real `notifyTerminatingSignals` registration is untested. Disputed, every fix worse: a test would deliver a real SIGTERM to the test process, which kills or disturbs the parallel tests that share it; the fake covers the handler goroutine, since the send times out without it.
- fix-proposal, LOW: a killed keeper makes the waiter wait the full bound. Not a defect: the bound is about 500ms and the wait gives up silently, as its comment says.
- fix-proposal, LOW: the same-producer, moved-history case emits no now-reads clause. Not a defect: F9's `fix` key allows exactly this.
- fix-proposal, LOW: F4's gate on `follow.spawned == nil`, and F8's generic `status_moved` expectation. Checked by the fork, no defect.

All four forks returned reports.

## Disputed

Both disputes are fork findings, stated under Self-check:

- Give-up path of the exit-mark wait: premise false, evidence the existing "not recorded" rows of `TestUntilStop_DeadLoopStopsLoopExited`.
- Real signal registration: every fix worse, evidence that a real SIGTERM reaches the whole test process.

## Deferred

Nothing deferred.

## Rulings

- F1: the parent's decision, creel #1150 from `ly:LOOM`, records that the load-turn retry cap and the idle time in `lym loom status` are left out of this run and filed as a board note.
  It asks for no change to the diff, and the decision record already holds the omission, so I made no change.
  I did not flag it to the parent.
