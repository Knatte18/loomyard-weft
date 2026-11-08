```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens, covering pairs of the record's decisions that act on the same state.

**Pairs checked**
- D1×D5, holder routing by channel map against the map entry being written before the invite: held.
- D1×D6, events for archived or reopened entries: held.
- D1×D2, a hint wakes the reader of the hub that owns the channel: broken (F3). Where the hints land does not match what D2's wake watches.
- D1×D2, a resync hint into every live hub's inbox after each connection: held.
- D1×Constraint `PATTERN-lym-files` ("paths built by `hubgeom.SlackGeometry`"), the holder writing into other hubs' inboxes and reading their maps: broken (F5).
- D2 thread cursor pruning against D2's thread-hint read: broken (F1).
- D2's 24 h `latest_reply` scan against D5's adopt path (cursor at newest message): broken (F2).
- D2's history read and thread read against D7 admitting `thread_broadcast`: broken (F4).
- D2×D6, a reopen keeps the cursors, then the resync or start read: held for top-level messages. Thread cursors pruned while the channel was archived fall under F1.
- D2×D5, a new channel's empty cursor against "cursors only move forward": held.
- D2 map writes against D5, D6 and `lym slack stop`'s map writes, all read-modify-write under the state lock: held.
- D2 wake on an fswatch event against Constraint `PATTERN-daemon-wakeups` (one-second floor): broken (F6).
- D3's user-token bound against D5 step 2's reopen, D6's reopen and D8's `is_archived` reopen: held. Each one is an own-map entry or an own D4 name that no other map holds.
- D3's three-token check against D10 setup: held.
- D4 `prime_beam` against D5's map key and D8's routing: held.
- D5 "every listed beam's channel when presence becomes slack" against D7 `away` in any channel and D9: held.
- D5's ensure on first mail against D6's "unlisted sender posts to the prime's channel" and D8's routing: held.
- D6 stop (relay ended, then archive from the CLI) against D5's presence-slack ensure and dobby's restart: held. Presence is terminal, so nothing restarts.
- D6's archive at each wake against D6's "a restart archives nothing on its own account": held.
- D8 `is_archived` and `channel_not_found` re-posts against the existing per-message `refusalCap`: held, capped at 5 per message.
- D7's operator and bot guards against D1, where the holder is a different hub's relay: held. The woken relay applies its own setup's operator.
- D10 manifest scopes against every call D3, D5, D6, D7 and D8 make: held.
- Acceptance criteria: the D2 "no wait beyond its hint's file event" bullet depends on F3, and the D2 "forwarded once" bullet is contradicted by F1 and F4. The others held.

**Findings**

F1 — BLOCKING, design, D2, thread cursor pruning against the thread-hint read.
A thread cursor is pruned after 24 h without a reply. A later operator reply in that thread then makes the relay read `conversations.replies` "after its thread cursor", and no cursor is left. The relay reads the whole thread and forwards every earlier operator reply again. That contradicts D2's rationale ("a message both hinted and caught up is forwarded once") and its acceptance bullet. D6's reopen hits the same case when a thread's cursor was pruned while its channel was archived. D2's stated bound only covers replies lost during a gap. It does not cover old replies replayed by a live hint.
Evidence: `_lym/discussion/decision-record.md:72`, `:79`, `:87`, `:90`, `:321`.

F2 — MEDIUM, design, D2×D5, the adopt path.
D5 adopts a live channel "with its cursor at its newest message", so the record means not to replay history. D2's resync and start read also takes every top-level message of the last 24 h whose `latest_reply` is past its thread cursor, "or that has no cursor". An adopted channel has no thread cursors, so the first start or resync after adoption forwards every operator thread reply of the last 24 h as new mail.
Evidence: `_lym/discussion/decision-record.md:80`, `:154`.

F3 — MEDIUM, consistency, D1×D2, where hints land against what the wake watches.
D1 writes hints into "that hub's inbox directory under `.lym/slack/`", which reads as a subdirectory. D2 wakes on "an `internal/fswatch` event on its Slack directory". `fswatch.Watch` adds the one directory and is not recursive, so a hint file in a subdirectory gives no event. The relay would then wake only when the outbound read's wait ends, up to 60 s. That breaks D2's acceptance criterion and the Goal's "within about a second". The record should name one directory for both.
Evidence: `_lym/discussion/decision-record.md:40`, `:83`; `internal/fswatch/fswatch.go:32-43`; `internal/slackengine/outbound.go:25`.

F4 — MEDIUM, design, D7×D2, `thread_broadcast` forwarded twice.
D7 admits `thread_broadcast`. Such a message is in the channel's `conversations.history` and is also a reply in its thread. D2 reads both routes with separate cursors: the top-level read after the channel cursor, and the thread hint's read or the 24 h scan after the thread cursor. The same message is forwarded once per route. Nothing in D2 or D7 makes one route skip the other.
Evidence: `_lym/discussion/decision-record.md:73-80`, `:198`, `:346`.

F5 — LOW, design, D1×Constraint `PATTERN-lym-files`.
The holder runs in `internal/slackengine`. It must read the channel map of every hub `hubregistry.Live` lists and write into each one's inbox. The engine "is told its `Geometry` and derives no path of its own". `hubgeom.SlackGeometry` returns `slackengine.Geometry`, so the engine cannot import hubgeom without an import cycle. The record names no seam for another hub's geometry. The only seams it names are for the Socket Mode connection (D11) and the existing ones, so the constraint's "paths built by `hubgeom.SlackGeometry`" has no way to hold for other hubs.
Evidence: `_lym/discussion/decision-record.md:39-40`, `:280`; `internal/slackengine/doc.go:17-18`; `internal/hubgeom/slackgeom.go:21`.

F6 — LOW, consistency, D2 wake against Constraint `PATTERN-daemon-wakeups`.
D2 wakes the relay on any fswatch event in its Slack directory, and that directory holds the relay's own frequently written records. Each successful Slack call rewrites `relay.json` through `recordingSlack.stamp`, and every post and cursor write rewrites `outbox.json` and the map. So the relay wakes on its own writes, with no one-second floor stated. Other fswatch-woken loops in the background state "never within a second of a wake's start". The Constraints line names no floor and no name filter.
Evidence: `_lym/discussion/decision-record.md:83`, `:277`; `internal/slackengine/relay.go:125-133`; `pattern/PATTERN-daemon-wakeups.md:26`, `:93`.

Tool calls: none. Every finding rests on what the inherited context already held.
