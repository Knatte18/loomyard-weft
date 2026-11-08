```yaml
lens: guard-reach
outcome: reported
```

Guard-reach review of the Slack push-channels plan: for each guard, refusal, branch and test row the cards add, can a real entry point reach it, and does the guarded site recur unguarded?

**1. MEDIUM — Card 8 (socket-holder), D1's ten-second envelope limit: the limit cannot fire on a stall, so a stalled holder is left unguarded.**
- Card 8 says routing happens "all within `envelopeLimit` measured on the clock", and that routing "that passed `envelopeLimit`" is acknowledged and owed a resync. That check can only run after the map reads and the hint write return.
- The case the record says the limit exists for is a holder "stalled on a map lock or a hint write, which the ping does not see" (decision-record.md:58, :88). A read or write that never returns never reaches the check.
- Card 8 also routes inside the serving loop that runs the pings. A stalled route therefore stops the pings too, and every other hub's events stop with it.
- The test row only advances the fake clock 11 seconds inside a writer that does return. That reaches the after-the-fact branch, not a stall.
- Fix: bound the map reads and the hint write with a deadline, and run them off the serving loop.

**2. MEDIUM — Card 7 (slack-client-surface), new `TestRunRelay` rows: neither row can reach the stamp it asserts.**
- In card 7's tree, `newRelayRun` sets `h.mailbox.blockReadWait`, so `ReadWait` blocks until cleanup (relay_test.go:79-93, inbound_test.go:174-177).
- The first outbound read therefore never returns, and `runOutbound`'s `afterRead` (the new stamp site) never runs.
- That breaks `OutboundReadStampsLastHealthyWithNoSlackCall` and the ten-minute idle row, which needs reads to complete across ten minutes of the fake clock.
- `internal/slackengine_test#newRelayRun` is not on card 7. Only card 12 changes it to answer each read at once.
- Fix: put `internal/slackengine_test#newRelayRun` on card 7 under `**Edit:**`, so the mailbox answers each outbound read at once with no mail, or have each row clear `blockReadWait`.

**3. MEDIUM — Card 7, the idle-relay row (D1 "idle for over ten minutes … not a young death"): as written, it never reaches the lifetime branch.**
- `starter.ensure` with whileAway answers `OutcomeTerminal` unless presence is `slack` (start.go:183-190).
- It also judges a lifetime only when `LastSpawnAt` is set (start.go:214).
- The row names neither setup step. Run as written, it answers "no failure counted" whether or not the stamp works, so its assertion cannot fail.
- Fix: the row writes presence `slack` and seeds `LastSpawnAt` before the run, as `RelayLifetimeAtOrOverTheHealthyBoundResetsTheCount` seeds its state (relay_test.go:812-819).

**4. MEDIUM — Card 11 (outbound-routing), `channelFor` and the refusal cap: the dropped-message line has no channel to post to.**
- Card 11 counts a refused ensure (`errChannelRefused`) as a refusal toward `refusalCap`.
- At the cap, `refused` posts the dropped line "to that message's channel" (outbound.go:196-199 today).
- For a message whose own channel ensure was refused, no channel exists. A prime channel blocked by another bot's channel of the same name hits this on every message.
- No card says where that line goes, and no test row reaches it.
- Fix: name a fallback, the prime's channel or a `[slack relay]` mail to the hub's orch, and add the row.

**5. MEDIUM — Card 12 (relay-push-loop), the outbound wake: the "one goroutine" guarantee is unguarded on the outbound path.**
- D2 requires one goroutine to drain the whole inbox, "so two wakes never read one channel at once" (decision-record.md:129).
- Card 12 has the outbound loop's `afterRead` "send an outbound wake" that runs `retryOwed`, `inviteOperators`, the retire retry and the prime ensure. It does not say that work runs on `wakeLoop`'s goroutine.
- Run on the outbound goroutine, `retryOwed` reads a channel while `wakeLoop.drain` reads it. The same goes for the inbound's unsynchronised state: the owed set, `failingTS` and the episodes.
- Card 12 also counts the outbound wake toward the one-second floor, which only works if it is serialised with file-event wakes.
- No row has a file event and an outbound wake coincide.
- Fix: state that `afterRead` only signals `wakeLoop`, which does the work and then runs the binary check.

**6. LOW — Cards 10, 11 and 12, the keeper's per-orch mutex (D5 "one ensure at a time"): it guards only if every caller shares one keeper.**
- The mutex, the cached `botUser` and the refused-retire memory live in one `channelKeeper` instance.
- Card 10 builds "the keeper" in `RunRelay`. Card 11 builds the outbound "with its keeper", and card 12 calls `ensure`, `inviteOperators` and `retire`. None of them says it is the same instance.
- A second keeper leaves the outbound's ensure free to race the start's or the wake's ensure of the same orch. That is two `conversations.create` calls and two entries.
- The concurrent-ensure row in card 9 runs inside one keeper only.

**7. LOW — Card 2 (socket-mode-client), `NewDialer`'s first-message read: a half-open connection during the dial hangs the holder.**
- The dial reads the `hello` with only the holder's context. The 30-second ping runs only while serving (card 8).
- A connection that goes half-open between `apps.connections.open` and the `hello` blocks the holder indefinitely, with no reconnect.
- D1's ping guard does not cover this site.

I made no further tool calls; everything I needed was already in context.
