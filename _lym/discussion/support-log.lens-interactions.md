```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked the pairs of decisions that act on the same state, then each acceptance criterion and constraint against the decisions it touches.

**Findings**

1. **MEDIUM** — D6 × D8 (`lym slack stop`)
   - The stop can leave a beam channel open.
   - D6 has `lym slack stop` archive every open beam channel *before* it ends the relay.
   - While the relay is still alive, D8 handles mail from that beam's orch: the post is refused with `is_archived`, so D8 reopens the channel and posts again.
   - The relay is then ended, and the channel stays open with its map entry marked open. This contradicts D6's promise and its acceptance criterion.
   - The order should be: end the relay, then archive. Or the relay itself archives as it shuts down.

2. **MEDIUM** — D2 × D7 (thread replies)
   - D2 claims a catch-up after each start or reconnect "covers every gap".
   - But `conversations.history` returns only top-level messages (and thread broadcasts), not thread replies.
   - D7 forwards thread replies, so a thread reply the operator sends during a lock hand-over, a reconnect or a relay start is lost for good.
   - Either D2's claim is bounded to top-level messages, or the catch-up also reads `conversations.replies` for threads active in the gap.

3. **MEDIUM** — D1 × D2 (inbox order)
   - D2 rests on "the inbox is handled in write order", so a resync marker's catch-up runs before the events written after it.
   - D1 writes one file per event into a directory and names no ordering key.
   - A directory listing or an fswatch event order does not give write order.
   - Without a sequence in the file name (or similar), an event written after the marker can be handled first. It then moves the cursor past messages the catch-up has not yet forwarded, and D2's skip rule drops them.

4. **MEDIUM** — D3 × D5 (user-token bound)
   - D3 bounds the user token to "a channel the hub's own map holds".
   - D5 step 2 reopens an archived channel found by name through `conversations.list` after `name_taken`. By construction, the map does not hold that channel.
   - The two decisions contradict each other.
   - The same path also lets a hub reopen the archived channel of an earlier, removed hub that had the same shortname on the same machine.
   - One fix: bound the step 2 reopen to the D4 name of this hub's own orch, and say so in D3.

5. **MEDIUM** — D5 / D9 × D1 / D2 (presence trigger)
   - D5 ensures every live beam orch's channel "when presence becomes `slack`".
   - Today the relay notices a presence change written outside it, by `lym slack away` from the CLI, through `presenceChanged` on each poll.
   - D2 removes the poll, and the record names only the inbox `fswatch` and the outbound wait as wakes.
   - So the CLI's `away` reaches D5's trigger only at the next outbound wake, up to a minute later, unless the relay also watches `presence.json`. The record never says it does.

6. **LOW** — D5 × D6 ("live orch" against "listed beam")
   - D6 archives when `slackengine.Hub` no longer lists the beam.
   - D5's trigger and D6's reopen speak of a "live" beam orch, and no definition of live is given.
   - After `lym beam stop`, the beam is still listed but its orch is down. Under the archive rule its channel stays open, while under the "live" wording its channel would never be ensured.
   - Pick one definition for both decisions.

7. **LOW** — D6 (failed beam read)
   - The archive-on-wake rule does not say that a failed `Hub.Beams` read is not "no longer listed".
   - Read naively, one failed read would archive every beam channel.

8. **LOW** — D1 × D5 (map write race)
   - D5 writes the map entry in step 4, after create and invite.
   - A message the operator writes in that window is routed by D1's holder to no hub, then acknowledged and dropped.
   - Step 4 then sets the cursor at the channel's newest message, which is after that message, so D2 never recovers it.
   - Writing the entry before the invite, with the cursor taken at create, closes the gap.

9. **LOW** — D2 × D5 / D6 × D8 (map writers)
   - The inbound handler (cursor), the outbound path (ensure, reopen, `channel_not_found` re-ensure), the wake archive and `lym slack stop` (another process) all rewrite `channels.json`.
   - The record does not say the writes are read-modify-write under the file's lock, so a lost update (such as a cursor rolled back) is left open.
   - This is borderline plan-level; I flag it because D2's dedupe depends on the cursor never moving backwards.

10. **LOW** — D1 × `PATTERN-daemon-wakeups` constraint (inbox drain)
    - The inbox is drained only on an `fswatch` event.
    - If an event is missed (watcher overflow, or a file written while the watch is being set up at start), the file waits until the next resync.
    - The fix is to state that each wake, including the outbound wait's, drains the inbox.

**Pairs checked**
- D1 × D2, resync marker against routing order: broken (3).
- D1 × D2, lock hand-over gap covered by the new holder's markers: held.
- D1 × D5, holder routing against when the map entry is written: broken (8).
- D1 × D6, the holder hub's `lym slack stop` releases the lock and the next relay takes over: held.
- D1 × D11, socket seam: held.
- D2 × D7, catch-up against thread replies: broken (2).
- D2 × D5 / D6 × D8, concurrent map writers: unstated (9).
- D2 × D10, old `cursor.json` and setup channel ignored, new map seeded at newest: held.
- D3 × D5, user-token bound against reopen after `name_taken`: broken (4).
- D3 × D6, unarchive adds the operator, the user token invites the bot, and D5 step 3's `already_in_channel` counts as done: held.
- D3 × login confirm of `xapp` through `apps.connections.open` against D1's one connection (it opens a URL, not a connection): held.
- D4 × D8, routing by sender against the names: held.
- D5 × D6, live against listed: ambiguous (6).
- D5 / D9 × D1 / D2, presence trigger with no poll: broken (5).
- D6 × D8, stop's archive against `is_archived` reopen: broken (1).
- D6 × D8, a gone beam's mail goes to the prime channel before any reopen: held.
- D6 × `Hub.Beams` failure: unstated (7).
- D7 × D9, presence commands in any channel: held.
- D9 × `SetPresence` notices: held.
- Acceptance criteria D1 to D11 against their decisions: held, except the D6 stop criterion (1), D2's "not forwarded again" with no ordering key (3), and D5's presence-trigger criterion with no presence wake (5).
- Constraint `PATTERN-daemon-wakeups` against D1 and D2: held, except the inbox drain (10).

No further tool calls were made.
