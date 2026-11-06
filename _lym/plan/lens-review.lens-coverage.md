```yaml
lens: coverage
outcome: reported
```

Coverage lens: every decision, criterion and card branch checked against the planned test rows. Findings below, defects only.

**BLOCKING**

1. **D3 steps 3 and 6, card 7 (`heldLocked`), card 3 (`Message` fields): the duplicate check has no receiver to compare.**
   - D3 compares the delivery's receiver with the held message's receiver, but card 3 adds only `SendingHub`, `SendingID`, `Trusted` and `RemotePosted`.
   - The only receiver left is `Holder`, which changes on a forward (`internal/creelengine/fold.go:267-271`, `eventForwarded`).
   - So a stored message forwarded while its `accepted` was lost gets a false id clash on the retry. The sender returns a message the receiver holds, breaking "a retry stores nothing twice" and D2's lost-`accepted` rule.
   - No row forwards between the two deliveries.

**MEDIUM**

2. **Confirm failure, card 7 step 4 and step 6:** no row reaches a failed, timed-out or other-word confirm.
   - The rows are the journal-fault row, confirms answering no, and the two-way delivery.
   - None has the sending daemon down at `confirm`, so nothing pins that the confirm starts no daemon (the operator's added decision) or that a failed confirm is transient and writes no drop event.
   - A regression here would drop genuine mail.
3. **D3 step 5 tuple:** only the `body hash` mismatch has a row.
   - No row delivers a real pending id with a different sender, which is the "claiming another strand" forgery in D3's bound. Receiver, reply-to, mark and post time have no rows either.
   - The first card 7 row ("a sender the sending daemon never sent") passes on a missing id alone.
4. **D3 step 2 and D1's bound ("mail from either is refused at the receiver"):** card 7's refusal row covers an unknown sending shortname only. No row covers a shared shortname (none or several), though `creelfake.Hubs` can answer it.
5. **D3 step 3 order ("Only then does it apply its receiver rules"):** the row "an unknown receiver after the duplicate check" passes with the order swapped. No row retries a stored message whose receiver has since left and expects `accepted`.
6. **Card 7 wiring guard contradicts card 4.**
   - Card 7 calls a `deliver` to a nil-seam daemon "a fail-loud wiring guard no entry point reaches in production".
   - Card 4's `serveDaemon` leaves `Hubs` nil when `registryDir` fails, so a production hub reaches it and every mail to that hub gets a transient refusal for the hour.
   - No row covers that `serveDaemon` branch or this answer.
7. **Card 7 "A request with no `Mail` is refused":** a hand-written line reaches this, so it is no wiring guard, and no row reaches it. The same holds for the failed strand listing at step 3 (`OK: false`, transient) and the empty body in `validateStampedLine`.
8. **D5 "a change in the base file applies to the next arrival":** `TestHubDirectory` shows the seam re-reads, but no engine row changes `Hubs.SetTrusted` between two arrivals. A store that cached the list would pass.
9. **D2 "start cap resets":** no row has the reset when that hub's daemon answers a call, nor the reset on a sending daemon restart. The card 8 row covers only the cap holding at three.
10. **D2 "each start is logged and reaped":** the criterion is argued in prose (`spawnDaemon` "logs and reaps") and `creelfake.Hubs` does neither. `TestHubDirectory` has no `Spawn` row, so `hubDirectory.Spawn` passing `hubPath` and `logPath` to `seam.spawn` is unreached.
11. **D2 retry schedule:** no row pins the wait from one second doubling to a minute, the cap at what is left of the hour, or "a wake during it changes nothing". Only the return at the hour is asserted.
12. **D2 "a timeout ... retried":** no row has a receiver that accepts and never answers, so the `timeout` code and the 30-second call bound are unreached. The `hub-unreadable` code (geometry fails after the post) and a failed spawn have no row either.
13. **D2 "oldest first per receiving hub":** no row posts two messages to a blocked hub and asserts their order.
14. **Card 6 `Store.WaitReply` and `settledOutcomeLocked`:** no row reaches the outbound branches.
    - Those are resume after a dropped connection, a wait registered after a return, a wait registered after a reply, and a wait by a strand that is not the sender.
    - Cards 8 and 9 only end a wait already registered.
15. **Card 3 retention:** the only inbound row is an unread message.
    - `carriedMessage` also carries a read, chaseable message, and no row checks that `Of: eventInbound` rebuilds `ReadAt`, the chase clock and `Chases`. The existing branch for this is at `internal/creelengine/fold.go:427-433`.
    - `applyCarried` lists no outbound fields: `ReceivingHub`, `Answers`, `ReplyTo` and the mark go unstated, and the row asserts only the post time.
    - No row folds an original line and its carried line together for an outbound or inbound message.
16. **D4 reply through a chase:** `Store.Reply` swaps `original` for the chased message at `internal/creelengine/verbs.go:211-219`. No row replies to a chase of a message from another hub, so the swap into the outbound path is unreached.
17. **Card 9 third reply condition ("its sender is the delivery's receiver"):** this is a refusal branch D4 does not name, and no row reaches it. The planned refused row covers only the wrong hub.
18. **Card 5:** its Intent says a test writes an invalid board file, but the row list has none. A corrupt record has no row either, so both named refusals are unreached.
19. **Card 8 "counted across a daemon restart":** `rig.start` resets the clocks to `epoch` (`internal/creelengine/server_test.go:118`), so a restarted sending rig loses the elapsed time. The card lists no way to start at a later clock, so the D2 row cannot show the hour counted across a restart as written.
20. **Card 9 end-to-end row:** it needs `Options.Hubs` from `internal/creelcli#rig.startDaemon` (`internal/creelcli/cli_test.go:311`), which no card lists. Without it, no row reaches the real seam through a daemon, and `serveDaemon`'s wiring stays unreached.

**LOW**

21. **Card 8 `settleReturned`:** "a strand no longer listed gets none" and "a notice that fails to store is logged" have no row.
22. **Card 8 final call answering `refused` at the hour:** no row.
23. **Card 1 `Record`:** "an entry whose hub path a stat answers with anything but not-exist is kept" has no row.
24. **D4 reply by a forwarded holder across hubs:** no row; only "a forward to a local strand works".
25. **Card 7 row "stops the receiving daemon ... through the journal write fault":** `FailJournalWrites` sits on a `Store` that `Run` builds privately, and no card adds a way to reach it.

Tool calls beyond context:
- One Bash call: the `UpdateJSON` declaration (my earlier listing was cut off), the `TestWriteMessagesText` glyph, and the board's configured labels for card 10's note. All three check out.
