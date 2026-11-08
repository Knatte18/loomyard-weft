```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked every pair of decisions that act on the same state.

**Pairs checked**
- D1×D2, hint write against hint drain and removal: **broken**, see F1.
- D1×D2, resync hint after each connection against the start and resync reads: held.
- D1×D2, `thread_broadcast` channel hint against the rule that only the top-level read forwards it: held.
- D1×D2, re-exec after the outbound read against releasing the socket lock and waiting for it again: held.
- D1×D3, login's probe connection against one connection per machine: held, since an unacknowledged event is retried.
- D1×D3, holder's connection against a bot-token replacement at login: **broken**, see F3.
- D1×D5, map entry written before the invite against routing by map: held.
- D1×D6, events for archived entries: held, since an archived channel gets no messages.
- D2×D5, thread floor set at adoption against the start read's thread scan: held.
- D2×D6, cursors kept across a reopen: held.
- D2×D3, inbound reads of the map's channels after a move to a new app: **broken**, see F5.
- D3×D5 and D3×D6, the user-token bound against unarchive on `name_taken` and the invite after a reopen: held.
- D3×D8, the fail-closed map test against the `not_in_channel` and `is_archived` retries and the cap: **broken**, see F2.
- D3×D5, the missing-token notice to the prime's channel against when the map first holds that channel: **broken**, see F4.
- D3×dobby, `OutcomeNotConfigured` against `EnsureRelayWhileAway` and the restart cap: held.
- D4×D5, `prime_beam` and the name hash against the create step: held.
- D5×D6, ensuring a beam channel against routing an unlisted beam's mail to the prime's channel: held.
- D5×D6×D8, routing while a `slackengine.Hub` read fails: **broken**, see F8.
- D5×D9, presence `slack` ensuring listed beams, whether set by `lym slack away` or the Slack text `away`: held.
- D5×D10, operator invite against a changed setup operator: **broken**, see F6.
- D6×D1, `lym slack stop` archiving from the CLI after the relay ends, against a holder in another hub: held.
- D6, an archive Slack refused, against the outbound wait and the daemon-wakeups constraint: held.
- D7×D9, presence commands in any channel: held.
- D8×D6, `is_archived` reopen and `channel_not_found` re-ensure against the gone-beam route: held.
- D10×D3, a setup file carrying only the operator against `RequireConfigured` and an old `setup.json`: held.
- AC D2 "records wake nothing" against D2's rationale: **broken**, see F7.
- AC D5 "invited each time" against D5's decision text: **broken**, see F6.
- Constraint PATTERN-daemon-wakeups against D1 and D2: held.
- Constraint PATTERN-lym-files against D1, D2 and D5: held.

**Findings**

**F1 — MEDIUM, design, D2 drain against D1 overwrite (decision-record.md:44-45, 109, 113).**
D1 overwrites a channel's hint for each new event, and D2 removes hints, but the record never says whether the relay removes a hint before or after its Slack read.
- If it removes after the read, the holder can overwrite the hint for a newer message while the read is in flight.
- The removal then deletes that newer hint, so the newer message waits for another message, a resync or a restart.
- D2's rationale only covers a lost hint when a "next" message follows. With no next message, push latency is unbounded, which breaks the Goal of about a second.

Fix: remove or rename each hint before its read, so a write during the read makes a new file and a new wake.

**F2 — MEDIUM, design, D3 fail-closed against D8 retries (decision-record.md:171-172, 303-308; outbound.go:167-171, 184-187).**
- While any other live hub's map cannot be read, D3 forbids the invite or reopen that D8 makes after a `not_in_channel` or `is_archived` refusal.
- That local refusal is not a Slack refusal, so it counts toward no cap.
- `postMessages` returns at the first failed post and releases the read, so this hub's whole outbound queue stalls behind that one message.
- It stays stalled until the other hub's map is fixed or the mail passes `staleMailAge`, 24 hours.

D8 states no bound for this case. Fix: count the fail-closed refusal toward `refusalCap`, or post past it.

**F3 — MEDIUM, design, D3 token replacement against D1 and D5 in running relays (decision-record.md:152-153, 227; wiring.go:188, 198).**
- A login that replaces the bot token, and removes the app-level and user tokens it does not match, says nothing about relays already running.
- Today each relay reads its token once at start. The holder keeps the old app's connection, and every relay keeps the old bot token.
- D5's creator test, "this machine's bot user", then has no settled meaning: the stored token or the one in memory.

Fix: state that relays re-read `slack.json` or restart on a token change.

**F4 — MEDIUM, design, D3 missing-token notice against D5 map creation (decision-record.md:160-162, 219, 329).**
- The case D3 names, a re-exec onto this task's binary with only a bot token, is the upgrade from the old binary.
- On that upgrade no `channels.json` exists yet: the old relay used `setup.json`'s channel, and D10 says nothing reads that key.
- The relay exits before D5 ensures the prime's channel, so the Slack line is never posted.
- Presence stays `slack`, and the away operator hears nothing. The orch's notice goes back out through the dead relay.

Fix: post to the old setup channel, or ensure the prime's channel before the token check exits.

**F5 — MEDIUM, design, D3's new-app path against D2 inbound (decision-record.md:185, 188-189).**
- D3's rationale says "the invite brings the new bot back in". The only invite path is D8's refusal of an outbound post.
- Until an orch posts, the new bot is no member of the map's channels. The start and resync `conversations.history` reads have no invite path, and Socket Mode delivers no `message.channels` events for those channels.
- So a channel's inbound stays dead until some outbound post happens. A beam with no outbound mail never recovers.

Fix: invite the bot on a read refused with `not_in_channel` as well, under the same bound.

**F6 — LOW, consistency, D5 against its acceptance criterion (decision-record.md:223-236, 444).**
D5's invite of the operator, step 3, sits under "Ensuring a channel the map does not hold". The acceptance criterion says "The operator is invited each time".
Combined with D10's `setup --operator`, an operator changed later is never invited into channels the map already holds.

**F7 — LOW, consistency, D2 acceptance criterion against D2's rationale (decision-record.md:126-127, 419).**
The criterion says the relay's writes to "its records" wake nothing. The rationale says the relay's own presence writes do wake it, and `presence.json` is one of the hub's records.

**F8 — LOW, design, D6 and D8 routing while a `slackengine.Hub` read fails (decision-record.md:215, 254, 264, 295).**
"Listed" holds only for a successful read. When the read fails, the record does not say whether a beam's mail goes to the beam's channel or to the prime's.

**Tool calls**
1. Grepped `internal/state` for a temp file or rename, to see where a hint write's temp file would sit; nothing matched.
2. Located the file that defines `WriteJSON` (`internal/state/state.go`).
3. Grepped `state.WriteJSON`'s write steps, for the same reason. It did not settle where the temp file sits, so F1 rests on remove order alone.
