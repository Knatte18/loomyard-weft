# Lens review

Every lens fork returned a report.

## criteria-coverage

- CC1 (BLOCKING, card 11, D5 fresh launch): a failed first release failed `start` against the record — fixed as proposed: `withdrawFirst` answers a failed release, which `start` records in `State.FreshLaunchFailure`; only a failed first withdraw fails `start`; a `TestStart_NoStrandLaunchesAndSpawnsWatcher` row covers a failed first release.
- CC2 (MEDIUM, card 3, command outcome pin): fixed differently: the proposal to have the gate run the llm smoke was not made, since the session's gate rule refuses llm-tier runs; the card drops the claim that a doc comment records the run's outcome, keeps the smoke's assertion as the pin and states the llm tier runs only when the operator runs it.
- CC3 (MEDIUM, card 7, D7 pending fold): merged with BC7 — fixed as proposed: the fold moves into `revivePending`, pinned by the new `TestRevivePending` in `internal/tendercli`, with the budget raised.
- CC4 (MEDIUM, card 10, re-read before queueing): merged with BC4 — fixed as proposed: card 10 states a re-read finding idle or another strand queues nothing, and a `TestWatcher_CycleHold` row rewrites the state from the fake's events read and asserts nothing is queued.
- CC5 (MEDIUM, cards 2 and 3, orch-2 sender): merged with RR1 — fixed as proposed: the hold rows name `ly:orch-2` and EXTERNAL with the program-only refusal text, and the command-inject rows add `ly:orch-2`.
- CC6 (MEDIUM, card 10, zero-time Stop): merged with BC17 — fixed as proposed: a `TestWatcher_TurnEndCountsOnlyAfterSending` row asserts a zero-time Stop after `sending` confirms nothing.
- CC7 (LOW, card 6, batten and hibernate wiring signatures): disputed, every fix worse: posting from a wiring test needs a creel daemon, which `internal/creelcli/spawn.go:54` refuses to spawn under go test, so a wiring row would need a cross-package export of creelcli's process-global client seam; `TestNotifyPrime_SignsAndPosts` pins the signing over fakes.
- CC8 (LOW, card 6, stale GUID): merged with FE1 — fixed differently: rather than keeping "no strand recorded" for a GUID reed does not hold, `PrimeOrchStrand` keeps `Strand != ""` as the recorded test, matching batten's `orchStrandRecorded` (`internal/battencli/wire.go:503`) and loom's `primeOrchRecorded`, and addresses an unheld GUID by `<shortname>:orch` so creel decides; the overview's notify-answer decision says so.
- CC9 (LOW, scope, creelcli doc refusal code): fixed as proposed: card 5's `internal/creelcli/doc.go` change describes the typed refusal code.
- CC10 (LOW, card 11, other mail reminded at once): disputed, every fix worse: reminding after a release is creel's behavior, pinned in `internal/creelengine` (`Store.Release` wakes the receiver, `internal/creelengine/verbs.go` `Release`); asserting it from `lyx orch start` would need a real daemon in `internal/orchcli`, which `spawn.go:54` refuses under go test; the start rows assert the second release is sent.
- CC11 (LOW, card 3, dormant receiver): fixed as proposed: a `TestScheduler_Injects` row keeps a command inject queued for a dormant orch strand and types it once live.
- CC12 (LOW, card 11, resume-line strand): fixed as proposed: `start` hands `ClearResumeNoticed` the GUID recorded before the reset, and a row asserts only that entry's mark clears.

## branch-coverage

- BC1 (MEDIUM, card 10, strand with no color): fixed as proposed: an empty `StrandColor` queues no color inject, the color step advances and the bind color counts as done; rows in the two color tests.
- BC2 (MEDIUM, card 10, unconfirmed branches): fixed as proposed: `TestWatcher_StepOutcomes` rows for an unconfirmed `/clear`, pointer and `/compact`.
- BC3 (MEDIUM, card 10, bind color outcomes): fixed as proposed: `TestWatcher_StartTypesTheColorOnTheFirstIdleTickOnly` rows for an expired or refused one and an unconfirmed one.
- BC4 (MEDIUM, card 10): merged into CC4.
- BC5 (MEDIUM, card 10, release failure at idle): fixed as proposed: the card states the watcher returns to idle through `creelFailed` and the hold lapses; a `TestWatcher_CreelFailuresHoldTheCycle` row.
- BC6 (MEDIUM, card 10, phase-reset failure): fixed as proposed: a failed reset withdraw or release saves no reset and the next tick runs it again; a `TestWatcher_CycleHold` row.
- BC7 (MEDIUM, card 7): merged into CC3.
- BC8 (MEDIUM, card 4, withdraw waiters): fixed as proposed: `TestStore_InjectAndHold` rows register an `InjectWait` and a `PostWait` and assert expired and returned.
- BC9 (MEDIUM, card 3, validate refusals): fixed as proposed: one table of every invalid command in `TestStore_InjectAndHold`.
- BC10 (LOW, card 3, Interrupt refusal): fixed as proposed: a `TestDaemon_Protocol` row.
- BC11 (LOW, card 9, start read failure): fixed as proposed: a `TestRun_ReturnsWhenStrandGone` row.
- BC12 (LOW, card 11, first release and adopt failure): fixed as proposed: first-release failure row (now recorded, CC1) and a `TestStart_LiveStrand` row for a failed first withdraw on the adopt path.
- BC13 (LOW, card 11, retry clears resume line): fixed as proposed: a row in `TestWatcher_CreelFailuresHoldTheCycle`.
- BC14 (LOW, card 10, ClearResumeNoticed no-op): fixed as proposed: the reset row leaves an entry alone when the withdrawn message is not the resume line; card 11's row covers an entry owning another GUID.
- BC15 (LOW, card 5, outside-hub refusal): fixed as proposed: a `TestCreelCLI` row.
- BC16 (LOW, card 7, reviveOrch errors): fixed as proposed: `TestRevive` rows for both seam errors.
- BC17 (LOW, card 10): merged into CC6.
- BC18 (LOW, card 10, soft restart test premise): fixed as proposed: the card says the test restarts with the post absent from the sent list, and adoption is `TestWatcher_RestartAdoptsThePendingStep`'s.
- BC19 (LOW, card 1, shuttlefake malformed stamp): disputed, premise false: the branch is test-kit code, not production behavior; `internal/testkit/shuttlefake`'s package doc says neither fake asserts anything and the kit carries no tests, and the production stamp rule is pinned by `TestRunner_ReadEvents` over `sessionFakeEngine`.

## row-reachability

- RR1 (MEDIUM, card 2): merged into CC5.
- RR2 (MEDIUM, cards 1 and 10, split stamp row): merged with TE1 — fixed differently: the proposal to drop the shuttlefake method was not made, since the decision record's Scope requires it ("the fake engine in `internal/testkit/shuttlefake/shuttlefake.go` gains a `ParseSessionSignals`"); the unreachable orchengine row goes and card 10 names `TestRunner_ReadEvents` as the pin, since only `Runner.ReadEvents` joins stamp and payload.
- RR3 (MEDIUM, card 11, harness shortname): fixed as proposed: `newStartHarness` writes the shortname record, and a hub with none fails `start` as a fail-loud wiring guard.
- RR4 (LOW, card 9, pause test notice rows): fixed as proposed: those rows assert creel posts, and the held-notice row keeps only the cycle request.
- RR5 (LOW, card 10, reply-before-save row): fixed as proposed: `fakeCreel` acknowledges only when the handler returns nil, and the row fails the handler's save by making the state directory unwritable.
- RR6 (LOW, card 4, sending inject row): fixed as proposed: that row moves to `TestScheduler_Injects` with a held `SetSendFunc`.
- RR7 (LOW, card 7, concurrent fake): fixed as proposed: the fake journal is mutex-guarded.

## forced-edits

- FE1 (MEDIUM, card 6): merged into CC8.
- FE2 (LOW, card 4, CLI timeouts): fixed as proposed: card 4 lists `postCmd` and `readCmd`, which pass `--timeout` only when waiting.
- FE3 (LOW, card 10, typeColor arrow): fixed as proposed: `typeColor` re-signs to return `error`.
- FE4 (LOW, card 10, prompt.go reasons): fixed as proposed: `render` is listed and the card rewords its refusal and the file comment.
- FE5 (LOW, stale notice-queue comments): fixed as proposed for code: card 6 lists `landingshed#Deps` and `sheddriver#handBackRun`, and verify gains both packages; disputed for `docs/research/codex-feasibility.md`, premise false: it is a research report recording a feasibility study at its date, which the prose rule's measurement-report exception leaves standing.
- FE6 (LOW, card 3, log subject): fixed as proposed: card 3 lists `logCmd` and updates its help and the `# Log` doc paragraph.

## test-economy

- TE1 (MEDIUM, cards 1 and 10): merged into RR2.
- TE2 (LOW, card 6, unit rows in integration tier): fixed as proposed: the fake rows move to the new untagged `TestNotifyPrime_SignsAndPosts` (budget raised), the integration test keeps the real-hub `PrimeOrchStrand` rows and its header is updated.
- TE3 (LOW, card 11, retry test): fixed as proposed: the retry rows join `TestWatcher_CreelFailuresHoldTheCycle` and `TestWatcher_RetriesAFailedFreshLaunchWithdraw` is not created.
- TE4 (LOW, card 11, start test): fixed as proposed: the rows join `TestStart_NoStrandLaunchesAndSpawnsWatcher` and card 11 raises no budget.
- TE5 (LOW, card 10, end-to-end row): fixed as proposed: hold assertions move into the re-expressed `TestWatcher_HandoffCompleteClearsThenResumes` and `TestWatcher_ClearCycleReloadsSkillsThenPointer`.
- TE6 (LOW, card 9, helper test): fixed as proposed: `TestCreelSeam_FoldsTheJournal` drives `creelSeam.InjectOutcome` and `Sent` over a `fakeCreelClient` through the new `creelClient` interface.
- TE7 (LOW, cards 4 and 5, withdraw repeats): fixed as proposed for the protocol strand-refusal row, which is dropped; disputed for card 5's row, premise false: it is the only row reaching `GoClient.Withdraw`, the exported surface card 5 adds.
- TE8 (LOW, card 10, note file with no reply): fixed as proposed: a `TestWatcher_NoteReplyGate` row.
