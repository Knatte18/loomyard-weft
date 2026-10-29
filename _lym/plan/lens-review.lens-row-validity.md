```yaml
lens: row-validity
outcome: reported
```

Lens row-validity: whether each planned test row reaches its branch, would fail if the code were wrong, and is needed.

1. MEDIUM — Card 4, row `AMarkedWaiterIsRungWithTheBudgetedClaimAndItsAckJournalsItAndAnswersTheRing`: the "no bell" check passes even if Ack never answers the ring.
   The row sets a running turn and then expects 45 s of probes to type no bell.
   But a probe that reads a turn in progress already marks the ring answered on its own (`internal/creelengine#worker.wakeHookStrand`: `if unanswered && !idleness.NoTurn { store.markRingAnswered(...) }`).
   So the assertion cannot fail when Ack fails to answer the ring.
   Fix: keep the turn ended before the ring and register a second waiter after the Ack, as `ARingAnsweredByAClaimIsNotFollowedByTheBell` does in `internal/creelengine_test#TestScheduler_HookWake`.
   The second waiter blocks the grace bell, and the turn end before the ring stops it being ringable, so only the ring's answered mark holds the 30 s bell back.

2. MEDIUM — Card 4, row `ARingOutcomeNeverTakenIsReleasedWhenItsWaitIsCancelled`: the order it states fails even with correct code.
   The row says "a claim hands the mail out, and the bell follows 30 s after the ring".
   A claim that hands out mail marks the last ring answered (`internal/creelengine#Store.ClaimAtBoundary`: `mark.answered = true`), and if it is acknowledged no claimable set is left, so no bell can follow.
   Fix: assert the bell (≥ 30 s after the ring) first and the claim after it, as the sibling row `AReleasedRingClaimLeavesTheMailUnreadAndTheBellFollows` does.
   Alternatively, release the claim and say that it is released.

3. MEDIUM — Card 6, new `TestCreelCLI` subtest `hook-wait whose stderr refuses the lines exits 0 and leaves the mail unread`: it races the daemon's release.
   When the handler fails, the client closes the connection and `hookWait` returns.
   The daemon's `awaitAck` sees the EOF and runs `Reading.Release` on its own connection goroutine, after that.
   The next tool-boundary `hook-read` sends a non-waiting claim (`internal/creelcli#hookRead`, `VerbClaim`), which can still find #1 reserved and print nothing.
   Fix: have the row wait until the daemon has released the claim before running hook-read.
   One way is a `read --wait` style step: Release answers the holder's waiters (`internal/creelengine#Reading.Release`).
   Another is to poll `lym creel log`/inbox.

4. MEDIUM — Card 4, `TestDaemon_Protocol` row `AMarkedHookWaitWhoseClientLeavesBeforeTheAckLeavesTheMailUnread`: same race.
   "Close after the ring; a strand read hands the message out" only works if the read waits.
   Fix: name `Request{Verb: VerbRead, Wait: true}`, as the existing row `ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread` does (`internal/creelengine/server_test.go:337`).
   A plain read can come back empty before `awaitAck`'s Release runs.

5. LOW — Card 4, row `APauseReadAtTheRingRingsNothingAndKeepsTheWaiter`: the second half needs the hook removed, and the card doesn't say so.
   `SetIdleHook` stays installed until it is set to nil (`internal/testkit/creelfake/creelfake.go:195-205`), so every later Idle call pauses alpha again.
   The step "after unpausing, a new post rings that same waiter" then never rings.
   Fix: the card should state `SetIdleHook(nil)` together with `SetPaused(alpha, false)`, or make the hook one-shot.

6. LOW — Card 9, `TestStore_ClaimLog`: the card names a Debug capture that the kit does not offer.
   `logcapture.Capture` leaves verbosity at Warn and `CaptureVerbose` raises it to Info only (`internal/testkit/logcapture/logcapture.go:43-60`).
   The Debug rows (`paused`, `held`, `stop-cap`, `stop-count-unknown`, `nothing-claimable`) need `logger.SetVerbosity(2)` after `Capture` (`internal/logger/logger.go:421`).
   Fix: the card should name that call so the five nothing-handed-out rows see their lines; Capture's cleanup already resets verbosity.
   Its bell rows also need the deliver_test env helpers (`e.probeWaits`, `e.tick`, `e.typed`), which are not on its Uses.

7. LOW — Card 7, `TestStore_MailFlags` row `AnOwedEchoKeepsTheFlagUntilTheClaimThatShowsItIsAcknowledged`: the env it needs differs from what `newFlaggedEnv` gives.
   `newFlaggedEnv` scripts every strand as `LiveStrand`, never `HookStrand` (`internal/creelengine/flags_test.go:38`).
   A pane-mail strand is never rung, so the row must re-script alpha with `e.del.SetStrands(creelfake.HookStrand(alpha), ...)` before `e.start`.
   Fix: the card says "a hook strand on a flagged env" but should state the re-script so the ring is reachable.

8. LOW — Card 7, row `TheStopCapKeepsTheMailBackButNotTheEcho`: the card does not give the claims' `stop_hook_active` values.
   The six-then-seventh count only reaches the cap when the first Stop claim passes `StopHookActive: false` and the rest pass `true`.
   With `false` on every claim, the count resets each time (`internal/creelengine#Store.ClaimAtBoundary`: `if boundary.Stop && !boundary.StopHookActive { s.stopBlocks[...] = 0 }`) and the seventh claim still hands out mail.
   Fix: state the flags as the existing row `StopBlocksAreCappedAtSixPerTurnAndANewTurnStartsTheCountOver` does.

9. LOW — Card 5, rows in `TestDaemon_Lifecycle`: the card does not say what bounds the call if the code is wrong.
   Without the change, `CallWatching` would wait on `c.clock.After(reconnectWait)` before reconnecting.
   On a fake client clock that never advances, the row hangs instead of failing on its "one connection" count.
   Fix: run the call through `callAsync` and `result` (`internal/creelengine/server_test.go:169-190`), whose real-time deadline makes a wrong client fail.

10. LOW — Card 6, `TestHookWaitExit`: two rows repeat one branch.
    The rows "written and the confirmation refused (2)" and "written and the confirmation lost (2)" both reach `hookWaitExit` with `wroteLines` true and an error that is not `ErrAckUnsent`.
    The function cannot tell them apart, so one row is redundant.

Calls made:
- Read the build tags of `deliver_test.go`, `server_test.go`, `flags_test.go`, `verbs_test.go`, `cli_test.go` and `hookread_test.go`, plus `newFlaggedEnv`, `logcapture.go` and creelfake's `SetIdleHook`/`Idle`. My context lacked the tags, the flagged env's strand script, the capture kit's verbosity levels and how long the idle hook stays installed.
- Read `logger.SetVerbosity`, because my context lacked the verbosity count that maps to Debug.
