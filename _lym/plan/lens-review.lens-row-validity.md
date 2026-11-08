```yaml
lens: row-validity
outcome: reported
```

Row-validity review of the 12-card Slack push plan: whether each planned test row can reach its branch, fails if the code is wrong, and is needed.

**BLOCKING**

1. **Card 7, rows `OutboundReadStampsLastHealthyWithNoSlackCall` and the idle-relay row (D1 "idle over ten minutes").** These rows cannot reach the stamp in card 7.
   - `newRelayRun` makes every outbound `ReadWait` block until cleanup (`internal/slackengine/relay_test.go:82-93`; `fakeMailbox.ReadWait` blocks at `internal/slackengine/inbound_test.go:174-177`). So `runOutbound` never returns from a read, and `afterRead` never runs.
   - Card 7 does not list `internal/slackengine_test#newRelayRun` under Edit. The non-blocking mailbox arrives only in card 12.
   - The idle row also needs a recorded `LastSpawnAt`, presence `slack` and a passed `NextStartAt` before `Ensure` judges a lifetime (`internal/slackengine/start.go:202-220`). That means combining `relayRun` with `ensureHarness` (`relay_test.go:553-564`). No helper for that is listed.

**MEDIUM**

2. **Cards 8 and 12: the shared `fakeClock` makes their wait-sequence rows unassertable and non-deterministic.**
   - `fakeClock.After` returns at once and advances time (`inbound_test.go:57-70`).
   - Under it, the holder's 30-second ping loop and card 12's instant-answering outbound loop spin. Each pass appends to `recordedWaits` and moves the clock forward.
   - The following rows read one shared `recordedWaits`, which the other loops' waits interleave:
     - card 8: "the clock's recorded waits run 1, 2, 4 … 60 seconds, and back to 1";
     - card 12: "an event inside a second … waits the rest of that second";
     - card 12: "no clock wait between";
     - card 12: "the on-disk check, whose waits run 1, 2, 4".
   - Clock-based limits also drift during a relay row: the five-minute `retryWindow` (`internal/slackengine/inbound.go:35`) and the 24-hour thread prune.
   - In card 8, "owe B a resync hint, written at the next ping" races the ping select against the envelope read.
   - Neither card lists a manual clock or per-loop wait tagging.

3. **Card 11, existing row `MessageSlackRefusesFiveTimesIsDroppedWhetherOrNotTheReplacementLinePosts`.** Its `callCount() == 6` assertion (`internal/slackengine/outbound_test.go:155`) breaks once each post goes through `keeper.ensure`.
   - `ensure` invites the operator when the entry records none (card 9) and calls `AuthTest` / `FindChannel` on some paths.
   - Card 11 never says the harness map entries record `Operator: "U1"`.

4. **Card 9, rows for `name_taken` adoption, archived or foreign-creator refusal, and the `-2` rename.** These rows need channels seeded into `fakeSlack` with a name, `Creator`, `IsArchived` and `IsMember`.
   - Card 7's fake declares only `CreateChannel` / `FindChannel` over "created and seeded channels".
   - No seeding method is declared. `fakeSlack.add` adds messages only (`inbound_test.go:282-286`).

5. **Card 9, row "an `Invite` hook reading the map sees the entry already written".** `fakeSlack.onCall` takes no method name and runs on every call (`inbound_test.go:262-275`), so it cannot single out `Invite`. No per-method hook is listed on card 7 or card 9.

6. **Cards 10 and 11: `RelayDeps.LiveHubs` and the keeper's `LiveHubs` have no stated nil meaning.**
   - `newRelayRun` hand-builds `RelayDeps` (`relay_test.go:95-104`). Card 10 adds `LiveHubs`, but its Edit list does not include `newRelayRun`.
   - Card 11's `newOutboundHarness` builds `NewChannelKeeper` without saying what `LiveHubs` it passes.
   - A nil seam reached through `heldElsewhere` (join after `not_in_channel`, adoption) would panic.

7. **Card 5, row "a dial answering app `A2`" and the piped-login row ("one dial").** The `newDialer` seam returns a `slackengine.Dialer`, which yields a `Socket` that login closes.
   - In `internal/slackcli` this needs a fake `Socket`. `slackengine_test#fakeSocket` is in another package and cannot be used.
   - No `slackcli` fake socket or dial counter is declared.
   - The slackcli `fakeSlack` records path and authorization only, not the form (`internal/slackcli/cli_test.go:51-55,70-73`). So no row can show `bots.info` received the `bot_id` from `auth.test`, which the default `{"ok":true}` answer does not carry. A login that sent an empty `bot` would pass.

8. **Card 5, a refusal branch with no reaching row.** "A failed live-hub read after the write answers an error naming the hubs record" names no row, and neither does `tokenRefusal` for a refused `apps.connections.open`.

**LOW**

9. **Card 11, two rows repeat existing ones.**
   - "five reads … drop the message, the line posting to the message's channel" repeats `MessageSlackRefusesFiveTimes…` (`outbound_test.go:120-164`).
   - "the stale line posts to `C1`" repeats `MailOlderThanADayIsAcknowledgedUnpostedWithOneCountLine` (`outbound_test.go:208-220`).
   - Each should be a channel assertion added to the existing row, not a new row.

10. **Card 12, `TestSlackCLI` `Stop` row "the fake Slack saw no request".** The row only catches a Slack call made in `stopRelay` itself.
    - The harness fakes the `stop` seam (`cli_test.go:219-223`).
    - The production `slackengine.StopRelay` path is never exercised.

11. **Card 12, row "the binary check's `beforeExec` leaves the socket lock free and a failed exec takes it again".** The holder retakes the lock in its own goroutine, and the card names no wait or synchronisation for the assertion.

12. **Card 1, `documentedAnswers` "gains the documented example answer of each new method from docs.slack.dev".** This leaves a fetch of docs.slack.dev to run time instead of recording the answers in the plan (rubric: research step).

13. **Card 6, row name.** `PrintsPresenceRelayPauseSetupAndLastSlackCall` keeps its name while it now asserts `channels` and `last_healthy`.

No further tool calls were made; my context held every file read.
