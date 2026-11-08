# Support log — slack-v2

## Interview

### Turn 1 — exploration (autonomous; no operator)

- Board: `slack-v2` (T1) is the task.
  `slack-presence-auto` (F2, note) and `slack-login-prompt` (T3) touch it.
  `slack-login-prompt` has landed on `main` as `fd159ff2`.
  No `Decision for slack-v2:` note exists, and `lym loom peers` lists no parent commits.
- Three lookup subagents mapped the codebase and Slack's docs:
  - Relay today: one detached relay per hub.
    It polls one configured channel every 5–60 s, routes by an `@<slug>` prefix and posts `[sender] body` with an `(answer: @slug …)` hint.
    Its state sits in `.lym/slack/`; the bot token sits in `slack.json` in the credential dir.
  - Hubs: one creel daemon per hub; `internal/hubregistry` lists a machine's live hubs, and this hub's shortname is `ly`.
  - Slack: each Socket Mode payload goes to any one of up to ten connections.
    Unarchive refuses bot tokens and needs a user token.
    The docs do not say whether an archived name stays taken.
- Every pick is an auto-pick, recorded in the question ledger.

### Turn 2 — design settled

- The push route is one connection per machine through a lock (D1), with a catch-up at start and after every reconnect (D2).
- The three tokens are filed by prefix (D3).
- Channel names, the map and the channel lifecycle: D4 to D6.
- Routing by channel and by sender: D7 and D8.
- Presence stays per hub (D9), setup and the manifest are D10, and the WebSocket module is D11.

### Turn 3 — lens review

A web-fetch lookup settled two facts before judging.
`conversations.invite` takes the user scope `channels:write`, and `conversations.replies` takes `oldest`.
Socket Mode's docs name no keepalive.
Every fork returned a report.

- bound-breaker 2 (BLOCKING), forwarding inbox file text: fixed.
  D1's inbox now holds text-free wake hints, and the relay forwards only what Slack returns (D2).
- bound-breaker 3 (BLOCKING), no dead-socket detection: fixed.
  D1 adds a 30 s ping with a 10 s pong deadline.
- walkthrough 1 (BLOCKING), and bound-breaker 4, a retried event lost by the cursor skip: fixed.
  Reads from Slack after the cursor replace the skip (D2).
- bound-breaker 1 (BLOCKING), walkthrough 3, duplicate shortnames and name clashes letting two orchs adopt one channel: fixed.
  The uniqueness claim is dropped, D5 refuses a channel any live map gives another orch, and `prime_beam` is a name no slug can reach.
- interactions 3, walkthrough 2, bound-breaker 7, inbox order undefined: fixed.
  Hints carry no order, and reads from Slack after the cursor make order moot.
- walkthrough 9, resync markers piling up: fixed; one overwritten `resync` hint per hub.
- interactions 2, bound-breaker 5, thread replies missed by catch-up: fixed.
  Thread cursors and a 24 h `latest_reply` scan are added, with the stated bound.
- interactions 1, existing-paths, walkthrough 5, bound-breaker 6, stop archiving before the relay ends: fixed.
  The stop now ends the relay, then archives.
- interactions 5, existing-paths, walkthrough 4, bound-breaker 9, no presence wake: fixed.
  The fswatch covers the Slack directory, and every wake drains the inbox and checks presence.
- interactions 4, user-token bound against reopen-by-name: fixed; D3's bound now names the D4 own-name and no-other-map rule.
- interactions 6, bound-breaker 8, live against listed, and an unguarded beams read: fixed.
  "Listed" means named by a successful read, and a failed read archives nothing.
- interactions 7: fixed with the same change.
- interactions 8, bound-breaker 11, creation-window loss: fixed; the map entry is written before the invite, with an empty cursor.
- interactions 9, concurrent map writes: fixed; read-modify-write under the state lock, and cursors move forward only.
- interactions 10, inbox drained only on fswatch: fixed; every wake drains it.
- existing-paths, start check against `RequireConfigured`: fixed; D3 makes one shared check that refuses before the spawn.
- existing-paths, bound-breaker 12, the token key and upgrade: fixed; the bot token keeps the `token` key, and the missing kind is named.
- existing-paths, program mail with no channel: fixed; D5 ensures at the first mail routed to a beam from its orch or a program.
- existing-paths, setup channel readers: fixed.
  `status` lists the map, setup drops the membership check, and the relay's lines post to the message's channel (D8, D10).
- walkthrough 6, missing `users:read`: fixed in D10.
- walkthrough 7, bound-breaker 10, subtypes skipped before the file and thread rules: fixed.
  D7 admits `file_share` and `thread_broadcast`.
- walkthrough 8, bound-breaker 13, the bot already a member and the invite's user scope: fixed.
  `already_in_channel` counts as done, and the user scope `channels:write` is confirmed from the docs.
- bound-breaker 14, reconnect loop after short connections: fixed; the wait resets only after a connection that lasted a minute.
- board-scope 1, icon dropped although cheap: fixed; D8 sets a fixed `icon_emoji` per sender kind.
- board-scope 2, the login prompt's wording overlapping with `slack-login-prompt`: fixed; D3 states the prompt names the three kinds.
- leanness 1, the PATTERN change stated three times: fixed.
  The `PATTERN-agent-addressing` constraint holds the rule once, and the PATTERN changes bullet stays because the change must be declared.
- leanness 2, rationale arguing against unchosen alternatives: fixed; the clauses in D1, D9 and D11 are cut.
- leanness 3, leanness 4, Open risks holding non-risks: fixed.
  The archived-name risk moves to D5's rationale and the question ledger, and the acknowledgement-deadline fact moves to Constraints.
- leanness 5, Slack facts restated in Constraints: fixed; only facts no Decision cites stay.
- leanness 6, assumptions restating Decisions: fixed; each line now names its pick and its D-id.
- leanness 7, a Notes bullet duplicating D8: fixed; it is cut.
- leanness 8, the D9 sentence about `slack-presence-auto`: fixed; it is cut from the record and kept in the ledger below.

## Rejected alternatives

- One Socket Mode connection per hub relay: Slack hands each payload to one connection, so hubs would get each other's events.
- A connection per relay, with relays forwarding foreign events to each other: N connections against Slack's cap of ten, and every relay must know every hub's map.
- A new machine-wide Slack daemon process: a second process kind, with its own start, restart and record, for what a lock gives.
- Writing the pushed event's text into the inbox and forwarding it: any local process could plant operator text, and order and retries would need their own rules.
- Keeping the history poll as a fallback beside push: the operator wants no polling wait and low idle cost; D2's catch-up covers the gaps.
- Renaming an archived channel and making a fresh one in place of reopening: loses the history in place, which the operator asked to keep.
- The prime's orch on the bot's direct message: one machine's bot has one DM shared by every hub there.
- Presence per orch: the operator would set it once per beam.
- Joining `slack-presence-auto`: it is a separate note, and only the operator promotes a note.
  Its switch on an operator Slack message would fit a channel per orch unchanged, since the relay sees every channel's message before it forwards it.
- Making every beam's channel at its orch's start: fills the sidebar with channels never used.
- A hand-written WebSocket frame codec: more code to maintain than a small dependency.
- A `lym slack manifest` verb printing the manifest: a doc file serves the one-time app setup.

## Review rounds

_No rounds yet._

## Question ledger

- Socket Mode with several hubs: resolved (research). One connection per machine, held through a lock (D1).
- Can a bot token unarchive? Resolved (research). No; the user token reopens (D3, D6).
- Can the user token invite the bot? Resolved (research). Yes; `conversations.invite` takes the user scope `channels:write` (D10).
- Does Socket Mode keep the connection alive? Resolved (research). The docs name no keepalive, so the client pings (D1).
- Does an archived channel keep its name taken? Open, since the docs are silent; D5 handles both answers.
- The name scheme's first segment: auto-pick. The hub's shortname, per the operator's `ly-neve-prime` example.
- The prime's channel at relay stop: auto-pick. It stays, for its history.
- The prime's orch on the bot's DM: auto-pick. No, a channel; the DM is shared by every hub on the machine.
- Presence per hub or per orch: auto-pick. Per hub.
- Does `slack-presence-auto` join? Auto-pick. No.
- Beam channel history at teardown: auto-pick. Archived and kept, and reopened with its history when needed again.
- Mail for a channel that no longer exists: auto-pick.
  A gone beam's mail posts to the prime's channel, `channel_not_found` ensures the channel anew, and `is_archived` reopens it.
- When a beam channel is made: auto-pick. At the first mail routed to it, and for every listed beam on presence `slack`.
- Threads: auto-pick. Operator thread replies are forwarded.
- Files and images: auto-pick. Named in the forwarded text, never downloaded.
- Icon: auto-pick. A fixed emoji per sender kind.
- Public or private channels: auto-pick. Public.
- Is the user token required everywhere? Auto-pick. Yes, so the configured check has one rule.
- WebSocket client: auto-pick. `github.com/coder/websocket`.
