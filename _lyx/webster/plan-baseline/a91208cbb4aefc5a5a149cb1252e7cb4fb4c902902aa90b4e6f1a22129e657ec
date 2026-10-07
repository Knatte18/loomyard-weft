# Card 5 — hold-turn-end

**Edit:**
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/rundir.go`
- `internal/shuttleengine/spec.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/wait_test.go`
- `internal/shuttleengine/waiting_test.go`
- `internal/shuttleengine/attach_test.go`
- `internal/shuttleengine/attach_asking_test.go`
- `internal/shuttleengine/spec_test.go`
- `internal/shuttleengine/completionsignal_enforcement_test.go`
- `internal/orchcli/start.go`
- `internal/orchcli/start_test.go`
- `internal/orchcli/smoke_live_test.go`
- `internal/loomengine/discussion.go`
- `internal/loomengine/discussion_test.go`
- `internal/loomcli/driverspec.go`
- `internal/loomcli/driverspec_test.go`
- `internal/loomcli/wiring_test.go`

**Intent:**
D4 and D9 (engine side): a turn end without every output file holds the run instead of ending it.

The hold.
`Run.Wait` never finalizes on a Stop, a live ask, or an expired-shell turn end whose output files are not all present.
`pollEventsTick` and `expiredTurnEnd` stop returning `OutcomeAsking`: they report a held turn end, carrying its last message, its outstanding tasks (the expired shells for an expired-shell turn end, none for a plain Stop) and the events-file offset just past the turn end's line, and Wait logs the hold at Info and keeps polling the same agent.
That offset is computed once, in `pollEventsTick`, from the batch's start offset and the classified last event's `Raw` line, and `recordWaiting` keeps it with the waiting turn end, so `expiredTurnEnd` reports the same offset later, on a tick with no new bytes.
The run ends only as today: done (every output file present at a turn end, the gate evaluated through `handleGatedBoundary`, which is unchanged), died, timeout or a mechanism failure.
The hold adds no negative outcome, so `PATTERN-completion-signal` is untouched; update the tripwire test's comments that describe `expiredTurnEnd`'s asking answer, and leave its audited counts as they are.
`finalize` no longer writes `AskingOffset`, and `RunState.AskingOffset`'s doc says it is read for legacy records only and never written.
The `OutcomeAsking` constant stays in this card, unreturned, so callers still compile; card 12 removes it.

What the hold lets through, and its bound: an agent's question or status turn end no longer halts its run.
A hold never extends `run.deadline`, so a held run is bounded by its caller's own deadline and by the liveness check, which still classifies a dead pane.
Until card 6 adds the notifier, every hold is silent; once it lands, a run with no notifier still holds silently to its deadline, where an asking stop used to end it at once.

`Spec.AwaitOperator` retires, since its behavior is now the default: remove the field, its Wait branch and its doc, and drop every assignment and assertion of it (`orchcli/start.go` and its two tests, `loomengine/discussion.go` and its test, `loomcli/driverspec.go`'s comment and `driverspec_test.go`, `loomcli/wiring_test.go`).

Doc: rewrite the `wait.go` header paragraphs on AwaitOperator and the Done/asking split, the `pollEventsTick` and `expiredTurnEnd` doc comments, the `Result.LastAssistantMessage` and `Result` doc in `run.go`, and the `shuttleengine/doc.go` sections on outcomes, AwaitOperator and background waiting, to describe the hold and its bound (the deadline is the caller's own `Spec.Timeout`, `run_timeout_min` only where it is zero, so the bound differs per caller; each Attach starts a fresh deadline).

Tests, by rewriting every existing test that expects `OutcomeAsking` from `Wait`, `pollEventsTick` or an `Attach` that reaches `Wait` to the held reading instead of adding beside them, so the suite passes at this card's commit.
A test whose subject is not the asking reading itself ends the held run through the exit it is not about, the output file appearing (Done) or the injected deadline (timeout), and asserts that exit.
The tests to rewrite are found by searching every `_test.go` file of the package, not only the files named below, for `OutcomeAsking` and for a Stop or ask line with no output file, and each file the search hits is an Edit target of this card; the ones known today are:
`TestRun_Wait_Classification`, `TestRun_Wait_EventsHandling`, `TestRun_Wait_Finalize_PersistsOutcomeForEveryTerminalOutcome` and `TestRun_Wait_Finalize_OutcomeWriteFailure_StillReturnsClassifiedResult` in `wait_test.go`;
`TestRun_Wait_AwaitOperator_AskingNonTerminal` in `wait_test.go`;
`TestPollEventsTick_Waiting`, `TestPollEventsTick_ShellExpiry` and `TestPollEventsTick_LaterTurnEndListingExpiredShellEndsWithoutNewWait` in `waiting_test.go`;
the `BacklogEndsInAsk_ClassifiesAsking` subtest of `TestAttach_OffsetStartsAtZero` in `attach_test.go`, a `running` record whose backlog ends in a Stop with the output missing, which now holds;
and `TestAttach_AskingReentryStartsAtRecordedOffset` in `attach_asking_test.go`, which attaches a legacy `asking` record whose events grew, asserts `OutcomeAsking` from the `Wait` that `Attach` reaches, and asserts the `AskingOffset` that `finalize` no longer writes.
This card adapts that test minimally: the output file exists before `Attach`, so the post-offset Stop finalizes Done and the test asserts that outcome and the post-offset message, and the `AskingOffset` assertion goes; card 7 rewrites the test into its final form.
Rows that name `asking` as a Wait outcome are rewritten here, and card 12 only clears constant uses that no longer assert it.
`TestRun_Wait_AwaitOperator_AskingNonTerminal` becomes the held-run table over the package's fake reed and engine with an injected clock:
a Stop with outputs missing keeps polling and a later Stop with all outputs finalizes Done, also through a gate;
a held run finalizes died on a dead pane, Done when its outputs appear as the pane dies, and timeout at its unchanged deadline.
`TestPollEventsTick_ShellExpiry` carries the expired-shell rows: with outputs missing the expired non-awaited shell is reported as a held turn end naming that shell, and with outputs present it is reported done.
Delete the `AwaitOperator` rows of `spec_test.go`.

**ImpactSummary:** Wait no longer returns asking, so every caller's asking branch becomes unreachable until cards 9–12 remove them, and Spec.AwaitOperator is gone with its assignments in its callers' packages.
