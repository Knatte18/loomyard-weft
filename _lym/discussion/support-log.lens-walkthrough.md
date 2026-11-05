```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision run end to end on one concrete case. Five decisions miss their purpose on at least one case.

**Findings**

1. **BLOCKING — D4 (cursor) with D6 (Slack presence commands).**
   - Summary: messages that produce no creel post never advance the cursor, so every poll handles them again.
   - Case: the operator writes `@nosuch hi`.
   - Step by step: the poll returns the message. It is top-level and from the operator. The slug is unknown, so the relay answers in Slack and posts nothing to creel.
   - D4 says the cursor advances "after creel answers the post's id". No post was made, so the cursor stays put.
   - On the next poll, 5 s later, the same message is answered in Slack again, and this repeats forever. The bot's answer resets the backoff, so the loop runs at the 5 s floor.
   - The same loop hits a text past the size cap, and the `away`, `back` and `status` commands. `status` re-answers on every poll, and `away` re-answers every poll even though the notice goes out only on a change.
   - Skipped messages (bot posts, other users, threads) are not "handled" either. The cursor then never moves past the bot's own outbound posts, and the poll window grows with every post.
   - Fix: advance the cursor past every message the poll returned, whatever its outcome. The creel post must still come before that message's cursor write.

2. **BLOCKING — D3 with D4 and D5 (relay identity in creel).**
   - Summary: whether the relay may post and read as `<shortname>:SLACK` depends on which process started it.
   - Evidence, from `internal/creelengine/doc.go` (Sender identity): creel walks the caller's process ancestry. A process inside a live strand's pane tree *is* that strand. A program name is accepted only from an orch pane or from outside every pane.
   - Case: a non-orch agent, or a hand-added strand, runs `lym slack away` while the relay is down. D3 has `away` start the relay. If the detached relay is still in that pane's ancestry, creel reads it as that strand.
     - D4's post as `SLACK` is then refused.
     - D5's `read --wait` reads that strand's mailbox, not `SLACK`'s.
   - The same applies to `lym slack start` run from a task pane.
   - D3 also re-execs the relay on a binary change, which keeps its ancestry.
   - Fix: state that the relay is started so it sits outside every pane tree, as creel's own daemon start presumably does. Alternatively, have `away` and `start` refuse from a non-orch pane, and state that as a bound.

3. **MEDIUM — D5 (ack granularity).**
   - Summary: one creel read hands out every unread message, and the acknowledgement covers the whole read.
   - Evidence: in `doc.go` (Mail), messages "are reserved until the client acknowledges them on the same connection". The client exposes no per-id acknowledgement; I checked the `Client` methods in `internal/creelengine/client.go`.
   - Case: the read returns messages #1 and #2. #1 posts to Slack; #2 gets a 500.
     - If the relay acks, #2 is lost.
     - If it does not ack, #1 is posted twice.
   - Also, holding the connection open through D5's backoff retries (up to 60 s) keeps every message reserved meanwhile.
   - Fix: decide the partial-failure rule. For example, ack nothing and accept a duplicate of what already posted, which fits D5's "at least once" bound. Or read with a cap of one message, if the protocol allows it.

4. **MEDIUM — D2 (scopes).**
   - Summary: `lym slack setup` cannot run with the scopes the record names.
   - Case: the operator grants exactly `groups:history` and `chat:write`, as the D2 Review criterion's help text says.
     - `users.info` needs `users:read`.
     - `conversations.info`, and its `is_member` check, needs `groups:read` (or `channels:read`).
   - Both calls answer `missing_scope`, so setup fails on a correctly configured app.
   - Fix: add those scopes to D2 and to the Review criterion, or drop the two checks.

5. **MEDIUM — D3 (restart) against the Goal.**
   - Summary: nothing restarts a relay that dies while presence is `slack`.
   - Case: the operator sets `away` and leaves. The relay then crashes, for example on a panic or an OOM kill. The record leaves the reboot case unclear.
   - Only the prime `BeamUp`, `lym slack away` and `lym slack start` start the relay. None of them can run from the phone, because the relay is the phone's only path in.
   - The operator is cut off, and reminders and orch posts pile up in creel unseen. That misses the Goal of steering from the phone while away.
   - Fix: have something that already runs periodically, such as each dobby pass or the hub watchdog, ensure the relay while presence is `slack` and config is complete.

6. **LOW — D8.**
   - Summary: "the time the run entered that row, read from its status record" is asserted without saying which field.
   - Evidence: the status history entries carry `At` (`internal/shedengine/status.go`, history entry), so it is probably derivable. The record does not say so.
   - Bound: low risk, but it is a fact D8 rests on and the record leaves it unchecked.

**Per decision: case used, purpose met or not**

- D1, operator posts in the `lyt` channel: the `lyt` relay reads it and no other relay does. Met.
- D2, setup with the named scopes: not met (finding 4). Login from stdin with `auth.test`: met.
- D3, relay started from the operator's terminal or the prime bring-up: met. Started from a task pane: broken (finding 2). Crash while away: not met (finding 5).
- D4, top-level "how is T2?" from the operator: reaches `ly:orch` as `[slack] how is T2?`, and the cursor advances. Met. Unknown slug, size cap and commands: broken (finding 1).
- D5, orch replies with `--reply` to a `[slack]` message: the reply goes to `SLACK`'s mailbox and is posted as `[ly:orch] …`. The bot post carries `bot_id`, so D4 skips it and there is no echo loop. Met. Partial batch failure: open (finding 3).
- D6, Slack `away`: presence is set, the answer goes to Slack and the orchs get a notice. Met apart from finding 1's repeat loop. The resume pointer line: met.
- D7, orch reports a halt while presence is `slack`: it posts to `<shortname>:SLACK`, which works because `SLACK` has read, so it is a known receiver. Met. Bound: a pre-relay `SLACK` receiver does not exist, but presence cannot be `slack` without config, so this holds.
- D8, PR-Review waiting 31 min under presence `slack`: dobby posts as `DOBBY` and the relay posts it to Slack. Met, assuming the row's entry time comes from history `At` (finding 6).
- D9, the cobra subtree: met.

**Tool calls**

- grep of the creel client for the read and ack API: my context held only `doc.go`, which does not say whether the ack is per id.
- grep for status time fields and the status package location: needed to verify D8's "time entered row" source, which my context lacked.
- A second, narrower grep of `Client` methods and the shedengine status `At` field: the first grep missed both because the paths were wrong.
