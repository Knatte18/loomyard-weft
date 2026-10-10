# Plan-Review round 1 — fixer report

Every finding was valid against the code and the decision record, and every one is fixed.
The fix of F17 inserts a card before the old card 8, so cards 8 to 19 became cards 9 to 20.
This report uses the new numbers, with the old number in parentheses where a finding named it.
The old card files were removed and rewritten under their new numbers.
`lyx loom validate-plan` passes with informational `create-new-unit` notes only.

## Fixed

- **F1** (PATTERN overview over its cap): measured with `wc -c`.
  The review's numbers hold: the old rewordings saved 238 bytes and the new line added 245, which ends at 16382.
  The new card 8 now shortens eight lines, adding `PATTERN-comment-line-breaks` and `PATTERN-no-denied-recovery` and trimming the six original rewordings further, so it saves 527 bytes.
  The `PATTERN-failure-stop-visible` line is shorter, at 238 bytes.
  The Shared Decision `pattern-overview-cap` now states the measured byte count each card leaves: about 15848, then 15874, then 16112, all under 16200.
  The background files for the five existing entries already hold every clause their lines drop.
  The three entries that had no background file get one in card 8.
- **F2** (an arming refusal read as `interrupted`): card 11 captures each child's stdout.
  When a child left no record, `stdoutErrEnvelope` reads its stdout error envelope as the stop step's envelope.
  An arming refusal therefore stops as `error` with kind `bootstrap`, and the loop test table has a row for it.
  Card 9 (8) moves `ReportArmError` and `KindlessRefusal` into `internal/shedverbs`, so the `lyx loom step` pre-run reports arming errors the same way.
  `loomcli` cannot import `shedcli`, which is why the helper moved.
  Card 12's integration test now pins a missing-hub-config `bootstrap` stop end to end.
- **F3** (race at loop start): card 12 adds a start handshake.
  `spawnLoop` returns a channel that closes when the spawned process exits, and the spawning waiter treats the loop as starting until it closes.
  `runLoop` takes its lock through `lock.AcquireWriteLockWithin` with `loopLockGrace`, so a momentary probe from a waiter or from card 18's quiet probe never makes a starting loop exit.
  A test row covers a waiter that polls before its loop takes the lock.
- **F4** (trace paths joined inside `shedverbs`): `LoopSpec.TraceFiles` is told by the arming module.
  Card 10 (9) adds `logger.TraceFilesFor(dir, traceID)`, which matches the sink's `trace-<ts>-<id>-<pid>.log` names.
  Card 15 (14) fills the closure over `logger.TraceDir()`.
  Card 11 reads, copies and reports only told paths, and card 13's `newestActivity` stats them.
  `PATTERN-shed-verb-set.md` states that a child's trace files reach the loop told through `Spec.Loop`.
- **F5** (no way forward on the interrupted stops): card 11's `interruptedError` composes each cause's message with a way-forward clause and passes it to `WriteFailedStop`.
  Card 12 uses it for `loop-exited`, and card 13 uses it for `watchdog`.
  The busy dead-loop stop gets its own message with a way forward.
  Card 6 states the contradictory `--clear` message, its way forward and its refusal row.
  The tests assert the clause.
- **F6** (tests drive unexported helpers): card 6 tests through `pauseCmd` with `execEnvelope` and re-reads the status file.
  Cards 11 to 13 drive the loop through `stepCmd` with `--until-stop`.
  Card 9 tests the exported `shedverbs.ReportArmError`.
  The same gap was also fixed in card 15 (14): its interrupt-hook test is now subtests of `TestSpecFor_ScratchAndFrictionDir` through the filled spec, and that card's new halt test and budget raise were dropped.
- **F7** (a waiter that only listens for fswatch events): `waitLoopEnvelope` re-checks on its backoff timer, and an fswatch event only cuts the wait short.
  A test row has a loop that dies while the fake watch emits nothing.
- **F8** (unbounded hidden flag): card 12's Intent states the flag's bound.
  `runLoop` refuses to step when it finds a pid file and no envelope, so the flag can never overwrite a dead loop's record.
  A test row covers this.
- **F9** (Windows code never compiled): `GOOS=windows go vet ./internal/proc ./internal/gatecli ./internal/shedverbs ./internal/pairteardown` runs clean on this host.
  Cards 1, 2, 12 and 18 carry a package-scoped `**Verify:**`, with a Shared Decision `windows-bodies-cross-vetted`.
  Card 1's false sentence is corrected.
- **F10** (card 20 (19) re-sign): `decision`'s `logNotice` closure becomes a `refreshNotice` value.
  `reconcileOne` carries the re-sign arrow and returns the notice.
  `decide` keeps its signature.
  `pendingWrites` and `WritesDue` stay silent, and a test asserts it.
- **F11** (overlapping test): `TestArmFromSeed_MissingRunRefusalCarriesNoKindField` is renamed to `TestArmFromSeed_RefusalEnvelopeKinds`.
  It becomes the table for the new rows and renders through `ReportArmError` instead of `output.Err`.
  The separate `TestReportArmError` and the `shedcli` budget raise are gone.
- **F12** (no grace between SIGQUIT and SIGKILL): card 13 waits up to `quitGrace` after `Quit` through the `Sleep` seam, then kills.
  A test row covers a child that exits within the grace.
- **F13** (leftover pid file after a delivered envelope): once the loop lock is free, a waiter or invocation that delivers or retires an envelope also retires a pid file left beside it.
  A currency test row covers this.
- **F14** (kills not logged): card 13 logs the quit and the kill, card 12 logs `deadLoopStop`'s tree kill, and card 18 logs the loop, tree and job kills.
  Card 1 states that callers log, because `proc` cannot.
- **F15** (findings file in webster's scratch directory): card 19 (18) adds `shedrecipe.Env.RunScratchDir`, which loom's `wire` fills with `shedrun.ScratchDir`.
  It re-signs `NewPlanGate`, `NewReworkPlanGate` and `planGate` to take that directory, and lists every caller, including four test files.
  An empty directory falls back to today's inline Warn.
- **F16** (Windows `KillTree` and the unnamed job): `TreeRecord` drops the job identity.
  On Windows `KillTree` reports false, because the child's job closed with the loop's named job.
- **F17** (card 8 bundles three units): the eight rewordings are their own card, the new card 8 `pattern-overview-room`.
  The kind vocabulary and the D6 arming refusal stay together in card 9.
- **F18** (invariant landing before its waiter): the `PATTERN-failure-stop-visible` line and its background file moved to card 12, with the waiter and the dead-loop stop.
- **F19** (stale `start.go` header): card 16 (15)'s Intent names the file-header rewording.
  Check 33 (`redundant-file-target`) would flag the file path listed beside `startCmd`, so it is not listed as a target.

## Disputed

None.

## Deferred

None.
