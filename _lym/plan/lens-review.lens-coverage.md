```yaml
lens: coverage
outcome: reported
```

Coverage lens review of the slack-machine-bridge plan: every decision, criterion and Intent branch checked for a carrying card and a reaching row.

1. MEDIUM — D10 / card 10 and card 11: the poll-wait criterion is only partly reached.
   - The record's D10 criterion needs three things: a re-read held message counts as not new, a newly delivered bridge message returns the wait to five seconds, and a Socket Mode event wakes the poll at once but never within a second of the last wake's start.
   - Card 10's only wait row is "5, 10, 20, 40, 60 and 60 seconds while nothing is new, and 5 again after a post". It is planned before the reader exists.
   - Card 11 adds no wait row: no held-message re-read keeping the doubling, no reset after a delivery, and no row for the wake through `wakeLoop.wake` → `bridgePoll.wake` or its `wakeFloor`.
   - Evidence: `_lym/plan/10-bridge-sending.md` (wait row and the `bridgePoll.wake` Intent), `_lym/plan/11-bridge-receiving.md` (Routing paragraph; its row list has no poll row).
2. MEDIUM — D8 / card 11: several of `bridgeReader.handle`'s numbered branches are reached by no row.
   - Branch 1: this bot's own posts skipped. No row puts this bot's envelope in a bridge channel and checks it is not delivered.
   - Branch 3: another human's message, or one with no `bot_id`, skipped.
   - Branch 4: a bot message whose first line `ParseBridgeMessage` refuses is skipped. Record D5 says "a first line that does not parse is skipped"; card 7's rows test only the parser.
   - Branch 10: an envelope whose `to` is not the channel's local orch is skipped.
   - Branch 13: a lost connection on `DeliverMachine` holds the cursor, as against a refusal.
   - Evidence: `_lym/plan/11-bridge-receiving.md`, `handle` steps 1, 3, 4, 10 and 13 against the `TestBridgeReceive` row list.
3. MEDIUM — D6 in a bridge channel / card 11: one half of the criterion "approve <machine> is handled only by the relay whose peer it names" has no row.
   - No row checks that an `approve` or `revoke` naming this relay's own machine is left alone: nothing pinned, nothing answered.
   - The rows cover only an `approve` that the relay does handle.
   - Evidence: `_lym/plan/11-bridge-receiving.md` `handle` step 2 ("one naming this machine is left to the peer's relay"); record D6 criterion 3.
4. MEDIUM — Card 9: refusal branches of `runPeerCommand` have no rows.
   - A machine name failing `machinename.CheckName`.
   - `approve <m> <app id>` with no bot of that app id, which is refused naming the id.
   - `revoke` of a machine with no pin.
   - A Slack refusal or a failed record write going through `retryOrGiveUp`.
   - Evidence: `_lym/plan/09-approve-and-revoke.md` (`runPeerCommand` bullets against the `TestInboundRead` rows).
5. MEDIUM — Card 9 against card 8: `approve <m> <app id>` depends on `BotUsersNamed` with an empty name keeping every bot. Card 8 never states or tests that.
   - Card 8 defines `BotUsersNamed` as keeping bot users whose `name` equals the given name.
   - Read literally, an empty name matches no bot, so `approve <m> <app id>` would always answer "none".
   - No `TestAPI` row covers an empty name.
   - Evidence: `_lym/plan/08-bridge-records-and-calls.md` (`BotUsersNamed` bullet), `_lym/plan/09-approve-and-revoke.md` (`approve <m> <app id>` bullet).
6. MEDIUM — D3 `--wait-reply` / card 3: the re-entry path has no row.
   - `WaitReply` accepting a machine-outbound id, and `settledOutcomeLocked` answering with a reply that already arrived, are never reached by a row.
   - The record says `--wait-reply` works "as for an outbound message to another hub", and the other-hub path has its own row (`WaitReply_OnAnOutboundMessageResumes…`, `verbs_test.go:343`).
   - Card 3's only row says the waiting post's wait "stays open". Card 4 checks a return answering a wait that was registered at post time.
   - Evidence: `_lym/plan/03-machine-outbound-post.md` (`WaitReply` and `settledOutcomeLocked` Intent; `TestStore_Mail` rows).
7. MEDIUM — Card 4: three no-op branches have no rows.
   - `ReportMachineHold` for a settled message writes nothing.
   - `SettleMachineReturned` leaves a settled message alone.
   - `machineHourLoop`'s wait is never less than a second.
   - Card 10's rows need `IsDaemonRefusal`'s answered-refusal path (the settle entry is dropped when creel answers a refusal), but only the lost-connection and success cases have rows.
   - Evidence: `_lym/plan/04-machine-relay-verbs.md` (Intent bullets against the `TestStore_MachineOutbound` rows); `_lym/plan/10-bridge-sending.md` (`settleAccepted` paragraph against its restart row).
8. MEDIUM — Card 5: two `DeliverMachine` refusals have no rows.
   - Step 4: a line over the cap is refused.
   - Step 2: a sender address that fails the parser for reasons other than naming this machine (a bad machine part, a program name part).
   - Evidence: `_lym/plan/05-machine-inbound-deliver.md` (`DeliverMachine` steps 2 and 4 against the `TestStore_MachineInbound` rows).
9. LOW — Card 3: `postMachine` refusal 2, a `Names` error returned as is, has no row. `creelfake.Bridge` as planned (`NewBridge`, `Refuse`) has no way to make `Names` fail.
   - Evidence: `_lym/plan/03-machine-outbound-post.md` (refusal list and the `creelfake.Bridge` paragraph).
10. LOW — D4 fidelity / card 10: the record limits the retried create, and the card limits only the listing.
    - Record: "While a `bridge-name-taken` or `bridge-archived` hold lasts, the relay tries the create again at most once per bridge-poll ceiling".
    - Card 10: `ensureBridge` calls `CreateChannel` on every send and throttles only `FindChannel`.
    - Evidence: `_lym/plan/10-bridge-sending.md` (`ensureBridge` paragraph); decision record D4.
11. LOW — Card 10: three sending branches have no rows.
    - A post refused `channel_not_found` dropping the bridge-map entry (only the `is_archived` row exists).
    - A transport failure or rate limit counting toward nothing.
    - A `Refusals` entry dropped once take no longer returns its message.
    - Evidence: `_lym/plan/10-bridge-sending.md` (`sendOne` step 6 and the `send` paragraph).
12. LOW — D8 criterion "a resync and a delivery accept an envelope naming this machine's former name as this machine": only the delivery half has a row.
    - The row is "an envelope naming this machine's former name as `to` is delivered".
    - No resync row adds a channel whose newest envelope names a former machine name.
    - Evidence: `_lym/plan/11-bridge-receiving.md` (`TestBridgeReceive` rows).
13. LOW — D8 resync trigger: the record says "a relay also runs a bridge resync at its start and at each bridge-poll ceiling", and no row checks that timing. The resync rows say nothing of what starts the resync.
    - Evidence: `_lym/plan/11-bridge-receiving.md` (Routing paragraph against its rows).
14. LOW — D8 operand: the resync is narrower than the record.
    - Card 11 excludes channels "that no orch channel map of a live hub holds".
    - The record says "a channel no orch channel map of a hub of this machine holds".
    - Evidence: `_lym/plan/11-bridge-receiving.md` (Resync paragraph); decision record D8.
15. LOW — `PATTERN-tolerant-records`: the bridge-notice and bridge-post records never have the unknown-key survival tested.
    - `TestBridgeRecords` covers only the peers record and the bridge map, and card 10 adds no such row.
    - Evidence: `_lym/plan/08-bridge-records-and-calls.md` (`TestBridgeRecords`); `_lym/plan/10-bridge-sending.md` (records paragraph).
16. LOW — Card 12: two branches have no rows.
    - A refused rename or archive of a bridge channel leaving the entry open and marking the orch refused.
    - A refused bridge-channel operator invite mailing one notice per episode.
    - Evidence: `_lym/plan/12-bridge-channel-upkeep.md` (Intent against the `TestChannelKeeper` rows).
17. LOW — D3 and D8 old-binary criteria: card 2 adds no row for these and instead argues they already hold.
    - It points to the existing generic `TestJournal_Fold` row "an unknown key and an unknown event name are ignored", which uses `from-the-future`.
    - No row folds the actual `machine-*` event names or a carried `of: machine-outbound` through that path.
    - Evidence: `_lym/plan/02-machine-mail-journal.md` (the "A binary that does not know these events" paragraph).

No further tool calls were made.
