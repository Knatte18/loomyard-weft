# Lens review

Every lens fork returned a report.
Duplicates across lenses are merged, and each line names the lenses that raised the finding.

## Card 1

- test-economy, LOW: the unknown-guid row repeats `TestResolveLivePaneID` — fixed as proposed: the row is dropped and the card names that test.
- branch-coverage, LOW: no row fails reed's styled capture — fixed as proposed: `TestCapturePaneArgv` gains a failing `capture-pane` row per read.
- branch-coverage, LOW: `shuttlefake.Reed.CapturePaneStyled`'s fallback to `CapturePaneFn` is untested — fixed differently: the proposal was to add a test, but the fallback served no caller, so the fallback is dropped and the method answers `CapturePaneStyledFn` or an empty capture, as `CapturePane` does.
- forced-edits, LOW: the `io.go` header edit lies outside the listed member — fixed differently: listing the file beside `Engine.CapturePane` is refused by `redundant-file-target`, so the header edit is dropped, since the header still describes the capture family truly.

## Card 2

- test-economy, LOW: `TestPlainBoxCapture`'s reading rows belong in `TestIdleSession` and `TestInputBoxText` — fixed as proposed: those rows move there, and `TestPlainBoxCapture` keeps only rows on the answer's own shape, which no test reads.
- branch-coverage, MEDIUM: the OSC, `ESC[22m` and bare `ESC[m` branches of `parseStyledLine` are unreached — fixed as proposed: `TestPlainBoxCapture` gains an OSC 8 row and two faint-clear rows that fail if typed text is blanked.
- forced-edits, LOW: `settings.go`'s header lies outside `settingsDoc` — fixed differently: the proposal was to name the header as covered, but the member glyph does not cover it, so the card targets the file `settings.go` in place of `settingsDoc`.

## Card 3

- criteria-coverage, LOW: "creel prefers the short pane (card 7)" names the wrong card — fixed as proposed: it now names card 6.
- forced-edits, LOW: the short-pane sentence is wrong for Claude, and the orch watcher's `Draft` use and `internal/orchengine/doc.go`'s residual are not listed — fixed as proposed: the sentence says Claude never sets both, the Intent names the watcher's `heldReason`, and `//internal/orchengine/doc.go` joins the targets.
- row-reachability, LOW: the recordless draft row's capture would read idle on `readinessEngine` — fixed as proposed: the row uses `BUSY\n❯ half a draft`.
- branch-coverage, LOW: no row fails `boxCapture` on the styled engine — fixed as proposed: a recordless subtest row fails `CapturePaneStyled` and expects the error.

## Card 5

- criteria-coverage BLOCKING, branch-coverage MEDIUM: a guard reading built on `operatorActivityHold` swallows a failed activity query, so the clear goes out with no operator check — fixed differently: the proposal was a row only, but the code itself was wrong, so the guard reading asks `reed.OperatorActive` directly and a query error fails it closed with `ErrInterruptClearFailed` and no clear key, and a row pins it.
- criteria-coverage, branch-coverage, MEDIUM: no row reaches a failed play of the clear — fixed as proposed: a row fails the `C-u` through `SendKeyErrQueue`.
- criteria-coverage, branch-coverage, MEDIUM: the not-ready and running-turn guard facts and an unreadable box are untested — fixed as proposed: `restoreReed.afterHead` draws `SHELL`, `RUNNING` or `NOBOX` after the key, `restoreEngine.Startup` reads `SHELL` as not ready, and a row pins each.
- branch-coverage, MEDIUM: no row fails the before-key capture — fixed as proposed: a row fails it and sends no key.
- branch-coverage, MEDIUM: no row reaches "no restored prompt after the key" — fixed as proposed: a row with no restored lines expects the `Escape` alone after two settles.
- branch-coverage, LOW: no row covers an engine without the restored-prompt clear — fixed as proposed: a plain `fakeEngine` row expects the `Escape` alone.
- criteria-coverage, MEDIUM: nothing pins that creel's interrupt reads go through the styled capture — fixed as proposed: `styledRestoreEngine` and a suggestion drawn before the key, where the styled engine clears and the plain one reads a draft.
- row-reachability, MEDIUM: the fakes cannot draw a pre-key draft, a pane with no running turn or a failing capture — fixed as proposed: `restoreReed` gains `before`, `idleBefore` and `suggestion`, and its captures consult the embedded `fakeReed`'s `CaptureErrQueue`.

## Card 6

- criteria-coverage, branch-coverage, MEDIUM: no row reads draft, then no reason, then draft — fixed as proposed: a `TestScheduler_Injects` row takes that sequence plus an `Idle` error and expects three `stalled` records.
- branch-coverage, LOW: `worker.remind` clearing the stall is unreached — fixed as proposed: a `TestScheduler_Reminders` row pins it.
- criteria-coverage, LOW: `TestHubDelivery` lacks the not-idle, no-reason probe — fixed as proposed: a `{}` row expects not idle with no stall.
- criteria-coverage, MEDIUM: the llm smoke test is assumed to pass without a run — fixed as proposed: card 6 carries `**Verify:** lyx gate test --tags llm ./internal/creelcli`.
- forced-edits, LOW: `creelfake`'s package doc goes stale — fixed differently: listing `creelfake.go` beside its `Delivery` members is refused by `redundant-file-target`, so the Intents of cards 6 and 8 carry the package-doc edit in prose.

## Card 7

- criteria-coverage, MEDIUM: no row asserts that the too-short wording names an allowed `lyx` command — fixed as proposed: the timeout row reads `StallTooShort` and asserts `stall_reason` names `lyx reed attach`.
- branch-coverage, MEDIUM: no row resumes a wait over a stalled receiver — fixed as proposed: a `TestDaemon_Protocol` row's `VerbWaitInject` gets the standing note at once.
- branch-coverage, LOW: the one-slot replacement and an expired outcome with no stall are unreached — fixed as proposed: a `TestScheduler_Injects` row over `Store.InjectWait` pins the replacement, and `InjectWaitReportsExpired` asserts no `Stall`.
- branch-coverage, LOW: "a client that cannot write it still gets its outcome" is unreached — fixed differently: the proposal was a row, but the claim was the card's own and not the record's, so it is dropped from the bound.
- row-reachability, MEDIUM: the timeout row can advance the client clock before any progress line arrives — fixed as proposed: rows run the verb in a goroutine and poll a `lockedBuffer` for the progress line before moving a clock.
- row-reachability, LOW: the synchronous `runCreelSplit` cannot run beside a clock move — fixed as proposed: it takes the stderr writer, and rows run it in a goroutine.
- forced-edits, LOW: rows that build a client directly would meet card 12's re-signed `NewClient` — fixed as proposed: the card says its rows build clients through `rig.client`.

## Card 8

- row-reachability, forced-edits, MEDIUM: the suspension row's `env.tick` waits out a real deadline per tick — fixed as proposed: the row uses `e.clock.Advance`, and closes the channel before cleanup stops the scheduler.
- criteria-coverage, LOW: the wake after the interrupt is unpinned — fixed as proposed: a row backs a worker off, ends an interrupt, and sees the inject typed with no clock move.
- test-economy, LOW: the `ErrClearFailed` row duplicates `Inject_InterruptQueuesAtTheHeadAndFailsOnOtherErrors` — fixed as proposed: it becomes a scripted error in that row.
- criteria-coverage, row-reachability, MEDIUM/LOW: no test reaches `shuttleOps.Interrupt`'s call to `Runner.InterruptRestoring` — fixed as proposed: a `TestHubDelivery` row builds a `shuttleOps` over a runner on `shuttlefake.Reed` with `restoringEngine` and `stepClock`, and expects the clear's `C-u` on a strand with no run record.

## Card 9

- criteria-coverage, branch-coverage, MEDIUM: no row reaches the TTL counted at `worker.finish` — fixed as proposed: a row holds the worker in a blocked send, posts a second inject, stops the scheduler, and expects its `aged` record's `consumed_s` to be 5, not 35.

## Card 10

- row-reachability, MEDIUM: the second `read` can race the daemon's release of the reservation — fixed as proposed: it retries under a real-time deadline while it answers nothing unread.

## Card 11

- test-economy, LOW: the reminder log row duplicates the backoff row's setup — fixed as proposed: the assertion joins `UnansweredReminderBacksOffToTheCeilingAndANewMessageResets`.
- branch-coverage, LOW: Release's failing strand listing is unreached — fixed as proposed: a row expects the read error and no journal line.

## Card 12

- branch-coverage, MEDIUM: the truncated-header bound and the pid-less lines before the header are untested — fixed as proposed: two excerpt rows pin them.
- branch-coverage, test-economy, MEDIUM/LOW: the spawn's fallback for a log that cannot be opened is unreachable under `go test` — fixed differently: the proposal was a row through the spawn, which `spawnDaemon`'s `go test` guard rules out, so the fallback moves into `openDaemonLog`, whose nil-file answer and Warn a row pins.
  The part of test-economy's finding that objects to testing an unexported helper is disputed, ground every fix worse: `internal/creelcli/spawn.go`'s `testing.Testing()` guard makes the helper the only surface a test reaches, so any other fix leaves the branch untested.
- row-reachability, LOW: a spawn answering pid 4242 cannot come through `rig.client`, and the rig's geometry lacks the log path — fixed as proposed: rows call `creelengine.NewClient` directly and set `r.geom.DaemonLogPath`.
- forced-edits, LOW: `internal/hibernatecli`'s `spawnDaemon` already has this shape — fixed as proposed: the card names it as the shape to follow.

## Card 13

- branch-coverage, LOW: the existing "daemon beside a live daemon" row now leaks the logger redirect and writes the header into its buffer — fixed as proposed: that row captures the logger and runs through `runCreelSplit`.
- test-economy, LOW: the `pidLineWriter` row tests a helper, not the verb — disputed, ground every fix worse: a line split across two writes depends on write boundaries no `creel daemon` run in `TestCreelCLI` can control, so a row through the verb cannot reach the buffering, and the card says so.
