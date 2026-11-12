```yaml
lens: interactions
outcome: reported
```

Task: run the interactions lens over the decision record and support log, checking each pair of decisions that act on the same state.

**Findings**

1. **BLOCKING — D8 with D6 and D4: one stray bot message can stop a bridge for good.** D8 holds the cursor at a message whose app does not match the `from` machine's pin, and only "a fixed pin" releases it. A bridge channel is a public channel, so any app with `channels:join` can join it and post a forged envelope `from=neve::ly:orch`. If neve's pin is already correct, the operator has no pin to fix. The cursor then stays before that message, and every later genuine message waits behind it. This defeats the purpose of pinning in D6. Evidence: D8 bullet "Otherwise it holds the cursor before it … or naming the mismatched app"; D4 makes the channel public with `conversations.create`. Fix: skip a message from a mismatched app once a pin exists, with a notice. Hold only while no pin exists.

2. **MEDIUM — D3 with D7: a post Slack accepted can still be reported as returned.** Suppose the relay posts, writes its outbox, and then the settle fails or the relay dies. After a restart, D7's outbox stops a second post, but nothing re-sends the settle. At the hour, D3's cross-hub-style return reports `outbound-returned` and posts a return notice for mail that was in fact delivered. The sender may then post again under a new id, and D8's dedupe, keyed on the sending id, does not catch that. Cross-hub avoids this by making one more call at the hour; the bridge has no counterpart. Evidence: D3 "message's hour, its return at the hour … work as for an outbound message to another hub"; D7 "outbox record keeps each bridge message Slack accepted". Fix: settle `remote-delivered` from the outbox on restart, before the hour.

3. **MEDIUM — D4 archiving with D7 sending: the peer keeps a dead channel id.** When an orch retires, its side archives and renames the bridge channel. The peer's bridge map still holds that channel id. D7 creates a channel only "when the bridge map lacks it", so the peer's next post goes to the archived channel. Slack refuses it five times and the message is returned. Nothing removes the stale entry or creates a new channel. A beam orch on the retiring side that later gets the same creel name would never be reached. Evidence: D4 "archives its bridge channels under `ArchivedName`"; D7 creates only when the map lacks the channel and returns after five refusals. Fix: drop the map entry on an archived-channel error and create again, or state that a retired orch's bridges are dead.

4. **MEDIUM — D8 resync with D5: reading the newest envelope can misroute.** A resync reads a `bridge-` channel "from its newest envelope". The newest message may be one this machine posted, so its `to` names the peer machine. The holder's routing rule, which requires `to` to be this machine, then fails, or picks the wrong local end. Evidence: D8 "adds a `bridge-` channel its map lacks, read from its newest envelope", and the holder routes only when "the `to` machine must be this machine". Fix: take this machine's end from whichever of `from` or `to` names this machine.

5. **MEDIUM — D3 or D10 button off with D7 pending mail: what happens is not stated.** D3 checks the button only at post time. If the orch's button is switched off while machine-outbound mail is pending, or the last button goes off and the relay ends, the record does not say whether the relay posts it, drops it as it drops orch mail today, or leaves it until the hour returns it with a creel code. D10 says an orch "sends only while its button is on", but no decision covers mail already queued. Evidence: D3 "refused at once … when the sending orch's slack button is off"; `internal/slackengine/doc.go` "drops the mail of an orch whose button is off"; D10. Fix: state the path and its return reason.

6. **LOW — D1 with D9: the untrusted stamp can never be used.** D1's parser refuses any `<machine>::` name whose name part is not an orch. D8 delivers through that parser, so a non-orch `from` never reaches creel, and D9's `(other machine, untrusted, …)` branch and its acceptance criterion cannot be reached. Evidence: D1 "the name part must parse as an agent name with role `orch`"; the D1 criterion refuses a non-orch name; D9 untrusted branch. Fix: either drop the untrusted stamp, or let the parser accept any strand name and enforce orch-only at post (D2) instead.

7. **LOW — D8 hold with D10 backoff: the poll may never slow down.** D10 resets the poll to five seconds "on a bridge message". A held message is read again on every poll. If a held read counts as a bridge message, the poll stays at five seconds for as long as the message is held. Evidence: D8 hold; D10 reset rule. Fix: count only newly delivered messages.

8. **LOW — D11 evidence with D6: `users.list` is not covered.** D6 calls users.list, but the D11 evidence list names only users.info and bots.info for `users:read`. The research did not fetch the users.list page. Fix: cite users.list's scope, or find candidates another way.

9. **LOW — D1 with D8 after a machine rename: messages are lost silently.** D8 skips a message unless its `to` names this machine's current name. Mail addressed to a former name (`machinename.Names`) is skipped without a notice. The open risk covers re-approving the new name, not this loss. Fix: accept former names in `to`, or post a notice.

**Pairs checked**

- D1 parser / D9 stamp: broken (finding 6).
- D1 own-name refusal / D8 to-machine check under a rename: broken in a minor way (finding 9).
- D2 orch-only / D3 refusal: held.
- D2 reply / D8 reply-to check: held.
- D3 post-time button check / D10 buttons: broken (finding 5).
- D3 hour return / D7 outbox: broken (finding 2).
- D3 hour / D7 unapproved hold, then `peer-unapproved`: held.
- D4 creation / D6 pin required for the invite: held.
- D4 create race (`name_taken`) / D6 adopt by the peer's creator: held.
- D4 archive / D7 map-driven send: broken (finding 3).
- D4 public channel / D8 mismatched-app hold: broken (finding 1).
- D5 envelope / D8 holder routing: held.
- D5 envelope / D8 resync: broken (finding 4).
- D6 machine-wide record / D7 and D8 release in other hubs' relays: held, bounded by the 60-second poll and the retry wake.
- D6 commands in any channel / D8 handling of operator text in a bridge: held.
- D7 notice in the orch channel / D3 button on at post: held.
- D8 hold / D10 backoff: unclear (finding 7).
- D8 button-off hold / D10: held.
- D8 dedupe / D7 at-least-once posting: held for the same id; broken across a repost after a false return (finding 2).
- D9 trust / D8 delivering only approved apps: held.
- D11 manifest / D6 and D8 calls: held except users.list (finding 8).
- Constraints (tolerant records, lym-files, config-layers, sole-parsers, daemon-wakeups) against D4, D5, D6, D8 and D10: held.
- Acceptance criteria: the D9 untrusted criterion cannot be reached (finding 6). The other criteria match their decisions.

No further tool calls made.
