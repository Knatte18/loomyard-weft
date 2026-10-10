# Plan-Review round 2 — fixer report

Every finding was validated against the code and the decision record and fixed; `lyx loom validate-plan` reports `ok: true` over the edited plan, with only `create-new-unit` informational findings.

## Fixed

- **F1 (BLOCKING), arming refusal under `--until-stop`.**
  Confirmed: `internal/shedcli/cli.go`'s `resolvePersistentPreRun` and loom's `armAt` arm in the pre-run, so the waiter's own arming refusal never reaches `runUntilStop`.
  Card 11 now creates `UntilStopFlag`, `ArmStop` and `ReportLoopArmError`, and edits both pre-runs: an arming error of a `step` carrying `--until-stop` writes `failed` through `shedengine.WriteFailedStop` and answers with `kind: bootstrap` plus the closed `loop` object (`stop` `error`, `steps` 0).
  A held loop lock or run lock writes nothing and stops `busy`, and an absent status file writes nothing.
  `TestArmFromSeed_RefusalEnvelopeKinds` gains those rows.
  Card 12's `TestLoopIntegration_ArmingRefusalStopsAsBootstrap` becomes a table whose new row runs the built binary's `lyx shed step <run-id> --until-stop` and asserts the `loop` key set and the status file reading `failed`.
  The `PATTERN-failure-stop-visible` background now names the arming refusal and that test.
- **F2 (MEDIUM), second waiter loses the envelope.**
  Card 12 gives each loop a loop id: the spawning waiter mints it with `logger.NewTraceID` and passes it on `--loop-detached`, and the pid file records it.
  The envelope file holds `loop_id` beside `envelope`.
  Retiring an envelope renames it onto a new delivered record, `shedrun.LoopDelivered` (card 3, filled by card 15).
  A waiter whose adopted loop is gone prints that record's envelope when it carries the same id, instead of reporting `loop-exited`.
  `TestUntilStop_SecondWaiterAdoptsTheLiveLoop` gains the two-live-waiters row.
- **F3 (MEDIUM), probe-held lock read as a live loop.**
  `runUntilStop` adopts a held lock as a live loop only once a pid file exists.
  A held lock with no pid file is re-read on the timer, and when the lock reads free the invocation decides again, which spawns.
  So a waiter reaches the gone rule only with an adopted id.
  The loser of a spawn race adopts the holder's id from the pid file.
  `TestUntilStop_LoserWaitsOnWinner` gains the instant-probe row.
- **F4 (MEDIUM), arming refusal's way forward.**
  Card 9's `ReportArmError` appends a generic way-forward clause to a `bootstrap` arming refusal whose text names none, and keeps an existing clause such as the hub-config one alone.
  The `step arming refused` row states that clause.
  The test's unknown-recipe row asserts it, and a new row asserts no doubled clause.
- **F5 (MEDIUM), teardown window.**
  Card 18 creates `seizeLoopLock`.
  After the kill, the teardown takes the loop lock and holds it until the reed session has ended.
  A loop re-taking the lock in between is killed again, under `runLockWait`.
  While the teardown holds the lock, a loop spawned by the still-live driver cannot take it and exits within the grace without stepping.
  The call-sequence table gains rows for the hold order and the re-kill.
- **F6 (LOW), in-flight records.**
  Cards 11 and 12 state that the `--until-stop` and `--loop-detached` branches return before `rec.begin()`.
  `TestLoop_RunsUntilStopAndPrintsOneEnvelope` asserts that the invoking process's own trace id has no in-flight record.
- **F7 (LOW), PATTERN entry shape.**
  The new line reads "Stopping a `--until-stop` loop on a failure with no other run-lock holder: the status file is non-running before its waiter returns. (test)".
  It is 3 bytes shorter than before, which leaves the overview at about 16137 bytes after card 12.
- **F8 (LOW), run-lock holder pid.**
  Card 18 creates `shedverbs.RunningStepPID`, which reads the newest unfinished in-flight step record.
  The run-lock wait's failure names that pid, or says the holder is unknown when no record is unfinished (a Go driver writes none).
  The failure no longer names the loop's pid file.
  The seize and wait failures name the way forward.
- **F9 (LOW), allowlist in the overview entry.**
  Card 11, the first card to grow the allowlist, now edits `PATTERN.md`.
  The shed-verb-set line gains "imports only its allowlist", 28 bytes.
  The Shared Decision `pattern-overview-cap` lists card 11 and the re-measured sizes: 15841, 15874, 15902 and 16137, all under 16200.

## Disputed

None.

## Deferred

None.
