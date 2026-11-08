# Decision record — slack-v2

## Goal

Replace the per-hub Slack relay's single polled channel with one channel per orch, woken by push.
Every hub on a machine shares that machine's one Slack app and bot.
An operator message reaches the orch of the channel it is written in within about a second.
An orch's posts reach its own channel under its own name, and lym makes, archives and reopens every channel itself.

## Scope

In:

- A Socket Mode push route that replaces the `conversations.history` poll, serving every hub on the machine over one connection.
- The app-level token and the operator's user token, stored beside the bot token through `lym slack login`.
- One channel per orch, the prime's and each beam's, named by hub, machine and orch.
  lym creates each channel, invites the operator, archives beam channels and reopens an archived channel with its history.
- Routing inbound mail by channel and outbound mail by the sending orch or program, with the sender shown as the post's username and icon.
- Operator thread replies forwarded, and attached files named in the forwarded text.
- `lym slack setup` reduced to the operator, and a documented app manifest listing every scope.

Out:

- `slack-presence-auto` stays its own board note; presence changes only by the routes it has today (D9).
- Downloading or relaying files and images, and any periodic poll.
- Any change to `slack-login-prompt`'s terminal reading of a token.

## Decisions

### D1 — One Socket Mode connection per machine, which only wakes the hub that owns the channel

Decision:
Each hub keeps its own `lym slack relay` process.
At most one relay per machine holds a Socket Mode connection: the relay holding the machine socket lock.
The lock is a file in the per-user directory `githubclient.CredentialDir` names, beside `slack.json`.
Every other relay blocks on that lock in a goroutine, and the first to get it opens the connection once the holder ends.

The holder opens the connection with `apps.connections.open` using the app-level token.
For each `events_api` envelope it finds the hub whose channel map (D5) holds the event's channel, reading the maps of every hub `hubregistry.Live` lists.
It writes a wake hint into that hub's inbox directory under `.lym/slack/`, then acknowledges the envelope.
A hint is a file named for the channel, or for the channel and thread for a thread reply.
A later hint for the same channel or thread overwrites it.
The hint carries no message text.
An event whose channel no live hub maps is acknowledged and dropped with a Debug log line.
A failed hint write leaves the envelope unacknowledged, so Slack retries it.

The holder sends a WebSocket ping every 30 seconds and reconnects when no pong comes within 10 seconds.
It also reconnects on a `disconnect` envelope or a read error.
Reconnects back off from one second, doubling to sixty.
The wait returns to one second only after a connection that lasted a minute.
A connection refused or disabled by Slack mails the holder hub's orch one `[slack relay]` notice per episode, naming `lym slack login`.

Rationale:
Slack hands each payload to any one of an app's open connections, and says to assume no pattern of distribution
(docs.slack.dev, "Using Socket Mode", read 2026-11-08: "each payload may be sent to any of the connections").
So one connection per machine is the only way one hub never gets another's events.
The OS lock hands it to the next relay when its holder ends.
A hint without text means a planted file can at most make a relay read its own channel from Slack.
It can never put words in the operator's mouth.
The ping catches a half-open connection after a suspend or a network change.
Slack's docs name no keepalive of their own (same page).

Bound:
The route wakes only the hub whose map holds the event's channel, and the woken relay forwards only what Slack returns for that channel (D2).
The ping is a bounded 30-second wake that spawns nothing.

### D2 — Slack's history is the source; a hint, a start or a reconnect triggers a read after the cursor

Decision:
Each channel's map entry carries a cursor, the newest top-level timestamp handled.
It also carries a thread cursor per thread, the newest reply handled.
A thread cursor is pruned once its thread has had no reply for 24 hours.
A relay reads a channel with `conversations.history` after its cursor at each of these:

- a hint for that channel;
- a resync hint, for each open channel;
- the relay's start, for each open channel.

A thread hint reads `conversations.replies` for that thread after its thread cursor.
A resync or start read also takes every top-level message of the last 24 hours whose `latest_reply` is past its thread cursor, or that has no cursor, and reads those replies.
After each connection it opens, the holder writes one `resync` hint into every live hub's inbox, overwriting any left there.
Cursors only move forward, and every map write is a read-modify-write under the map's `internal/state` lock.
A relay drains its whole inbox at every wake: an `internal/fswatch` event on its Slack directory, or the outbound read's wait ending.

Rationale:
Reading from Slack after the cursor makes order, retries and duplicates harmless.
For example, a hint lost for one message is covered by the read for the next, and a message both hinted and caught up is forwarded once.
The gaps push leaves are a relay not running, the lock hand-over and a reconnect.
A start or a new connection follows each of them, and its catch-up runs then.
Bound: a thread reply sent during a gap to a message over 24 hours old is not recovered.

### D3 — Three tokens in the machine credential file, filed by their prefix

Decision:
`slack.json` keeps the bot token (`xoxb-`) under its existing `token` key and adds the app-level token (`xapp-`) and the operator's user token (`xoxp-`).
`lym slack login` prompts for one token, naming the three kinds it takes, and reads it in whatever way `slack-login-prompt` lands.
It files the token by its prefix.
It confirms a bot or user token with `auth.test` and an app-level token with `apps.connections.open` before writing it.
It refuses any other prefix, naming the three it takes.

The relay start and `slackengine.RequireConfigured` apply one check: every one of the three tokens and the setup file present.
A machine missing any refuses before a relay is spawned, naming `lym slack login` and the missing kind.
So `EnsureRelay` and dobby's restart never spawn a relay that refuses at once.

The user token is used only for `conversations.unarchive` and for inviting the bot into the channel it reopened (D6).
Both calls are made only for a channel named this hub's own D4 name for one of its orchs, held by no other orch's entry in any live hub's map.
No token is ever config, an environment variable or a log line.

Rationale:
Socket Mode needs an app-level token (docs.slack.dev, `apps.connections.open`).
Slack refuses a bot token for `conversations.unarchive`
(docs.slack.dev, `conversations.unarchive`, read 2026-11-08: "Bot tokens (xoxb-...) cannot currently be used to unarchive conversations").
The operator asked to reopen an archived channel with its history.
The prefixes tell the kinds apart, so login needs no flag.
Keeping the `token` key lets an older binary on the machine still read the bot token.
Bound: the user token can act in any public channel the operator can reach, and only the two calls above, on the channels above, use it.

### D4 — Channel names: shortname, machine, orch

Decision:
A channel is named `<shortname>-<machine>-<orch>`.
`<shortname>` is the hub's shortname, `<machine>` is the `internal/machinename` name, and `<orch>` is `prime` for the hub's own orch or the beam's slug.
A beam whose slug is `prime` gets `prime_beam`, which the slug grammar `[a-z][a-z0-9-]*` cannot produce.
Any character of the machine name outside Slack's channel grammar (lowercase letters, digits, `-`, `_`) becomes `-`.
A name over Slack's 80 characters is cut to fit a suffix of `-` and the first six hex digits of the SHA-256 of the whole name, 80 characters in all.

Rationale:
The operator settled the order, repository then machine then orch, as in `ly-neve-prime`.
The hub's shortname is the repository's name in that example.
The machine segment keeps each machine's app on its own channels.
The sort keeps each hub's channels together, and each machine's apart under it.
A machine's hubs can share a shortname: `hubregistry.Live` lists them all, and only cross-hub mail refuses them.
So D5, not the name, keeps two hubs off one channel.

### D5 — The channel map and when a channel is made

Decision:
Each hub keeps `.lym/slack/channels.json`, written through `internal/state`.
It maps each orch name to its channel's id, name, cursors and whether lym archived it.
A beam counts as listed when a successful `slackengine.Hub` read names it.

The relay ensures:

- the prime's channel at its start;
- a beam's channel at the first outbound mail routed to it, from its orch or a program (D8);
- every listed beam's channel when presence becomes `slack`, seen at the next wake (D2).

Ensuring a channel the map does not hold:

1. `conversations.create` for the D4 name as a public channel, then the map entry at once, with an empty cursor so the first read starts at the channel's beginning.
2. On `name_taken`, `conversations.list` with archived channels included, to find the channel of that name.
   - When no live hub's map gives that channel to another orch:
     - an archived one is reopened (D6);
     - a live one the bot is a member of is adopted, with its cursor at its newest message.
   - Any other is refused, and the hub's orch gets one `[slack relay]` notice naming the channel.
     When another hub's map holds the channel, the notice names `lym fabric shortname` as the way forward.
3. `conversations.invite` of the operator, where `already_in_channel` counts as done.

Rationale:
The operator makes no channel by hand, and a channel appears only when it is first needed.
The prime's orch is live while its relay runs, so its channel exists from the start.
Writing the entry before the invite means the operator's first message in a new channel is already routable.
The map, not a name search, is the authority for an existing channel, so a list is read only when a create collides.
Slack's docs do not say whether an archived channel keeps its name taken (`conversations.create`, read 2026-11-08), and step 2 handles both answers.
Bound: two hubs sharing a shortname that ensure the same name in the same instant can both adopt it before either map is written.
That race needs duplicate shortnames, which creel already refuses for cross-hub mail.

### D6 — Beam channels archive when the beam goes, and reopen when it is needed again

Decision:
At each wake, the relay archives the channel of every beam its map holds open that is no longer listed, and marks the entry archived.
A failed `slackengine.Hub` read archives nothing.
`lym slack stop` ends the relay first, then archives every open beam channel of the hub from the CLI.
lym never archives the prime's channel.
A relay that dies and restarts archives nothing on its own account.

An archived entry is reopened when D5 would ensure its channel.
To reopen, the relay calls `conversations.unarchive` with the user token.
It then invites the bot with the user token, where `already_in_channel` counts as done, and marks the entry open.
The cursors stay where the entry left them.
Mail whose sender is a beam orch or program no longer listed posts to the prime's channel, under the sender's name.

Rationale:
The sidebar holds only listed beams and the prime, and the operator removes no channel by hand.
Archiving after the relay has ended means no live relay can reopen a channel the stop has just archived.
A bot token can archive but not unarchive (D3), and the user who unarchives joins the channel (docs.slack.dev, `conversations.unarchive`).
Keeping the prime's channel keeps the hub's history at hand.
A gone beam's mail lands in the prime's channel, without reopening a channel nobody will answer in.

### D7 — Inbound routing by channel

Decision:
An operator message reaches the orch the map gives for the channel it was written in.
The relay drops the `@<slug>` prefix routing and the `(answer: @<slug> …)` hint on outbound posts.
`away`, `back` and `status` work in any of the hub's channels and set or read the hub's one presence.
Each is answered in the channel it was written in.
An operator thread reply goes to the same orch as a top-level message.
The relay forwards a message of no subtype, `file_share` or `thread_broadcast`.
It skips any other subtype, a bot's message and anyone's but the configured operator's.
A message carrying files forwards its text with one line per file, `(file not relayed: <name>)`.
lym never downloads a file.

Rationale:
The channel tells the orchs apart, so the operator writes no prefix.
A thread reply to an orch's post is the natural way to answer it.
Naming a file tells the orch something was sent that it cannot read.

### D8 — Outbound routing by sender, posted under the sender's name and icon

Decision:
Mail to `<shortname>:SLACK` goes to the channel of its sender.
- A beam orch, or a program of the form `<shortname>:<slug>:<NAME>`, goes to that beam's channel.
- The hub's orch, or a program of the form `<shortname>:<NAME>`, goes to the prime's channel.
- `agentname.TaskSlug` and the existing program parse give the slug.

`chat.postMessage` sets `username` to the sender's creel name and `icon_emoji` to `:robot_face:` for an orch or `:gear:` for a program, through `chat:write.customize`.
The body is posted without the `[<sender>]` prefix.
The relay's own lines about a message, dropped, stale or refused, post to that message's channel.
A post refused with `is_archived` reopens the channel (D6) and posts again.
A post refused with `channel_not_found` drops the map entry, ensures the channel anew (D5) and posts again.
Each refusal counts once toward the refusal cap.

Rationale:
Every orch posts to the one address it uses today, and lym looks up the channel, so no orch knows its channel.
Username and icon are fields of the call D8 already makes (docs.slack.dev, `chat.postMessage`).

### D9 — Presence stays one per hub

Decision:
Presence stays the hub's one `presence.json`.
`slackengine.SetPresence` still notifies every orch of the hub.
`slack-presence-auto` is not built here.

Rationale:
The operator is one person in one place.

### D10 — Setup names only the operator; the app manifest is documented

Decision:
`lym slack setup --operator <user-id>` records the hub's operator after `users.info` confirms the user.
The `--channel` flag and the channel membership check go.
A `setup.json` written by an older binary keeps its channel key, which nothing reads.
`lym slack status` lists the map's channels, open or archived.

`docs/slack-app.md` (new) carries the app manifest and the steps to make a machine's app. The manifest sets:

- the app name `lym-<machine>`, with Socket Mode on;
- the bot event `message.channels`;
- the bot scopes `channels:history`, `channels:manage`, `channels:read`, `chat:write`, `chat:write.customize` and `users:read`;
- the user scope `channels:write`;
- the app-level token scope `connections:write`.

The slack bullet of `docs/overview.md` and the package documentation of `internal/slackengine` and `internal/slackcli` describe the new route.

Rationale:
lym makes the channels, so the operator names none.
`internal/state` keeps an unknown key, so an old setup file needs no migration.
The user scope `channels:write` covers both `conversations.unarchive` and `conversations.invite` (docs.slack.dev, both method pages, read 2026-11-08).

### D11 — The WebSocket client is `github.com/coder/websocket`

Decision:
`internal/slackengine` adds `github.com/coder/websocket` for the Socket Mode client.
The connection sits behind a seam the relay is told, as its `SlackClient` is today.
The tests feed envelopes through that seam, and one test runs the real client against an `httptest` server.

Rationale:
The Go standard library has no WebSocket client, and this module is context-aware and needs no further module.

## Constraints

- `PATTERN-agent-addressing`: only `internal/slackengine` calls Slack, over both the Web API and the Socket Mode connection.
  Operator text still reaches an agent only as creel mail from `<shortname>:SLACK`, and only as Slack returned it.
  The background rule's new text is in `## Notes for the plan writer`.
- `PATTERN-child-processes`: no new process kind; the relay keeps its double-fork start and its `runproc.HubOwner` record.
- `PATTERN-daemon-wakeups`: the inbound poll goes.
  The inbox and presence wake is an `internal/fswatch` event, and the lock wait blocks without waking.
  The 30-second ping spawns nothing, and the reconnect backs off from one second to sixty.
  The outbound read keeps its wait.
- `PATTERN-lym-files`: the hub's new files sit under its `.lym/slack/`, with paths built by `hubgeom.SlackGeometry`.
  The machine socket lock sits beside `slack.json` in the credential directory, which is machine state outside every hub.
- `PATTERN-tolerant-records`: the channel map and the token file are written through `internal/state`.
- `PATTERN-config-layers`: no config key is added; tokens, operator and channels are never config.
- `PATTERN-refusal-way-forward`: each new refusal of `login`, `setup`, the relay start and `RequireConfigured` names its way forward.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: the relay tests stay tier 0, with an injected clock and fakes for the Slack client, the connection and the mailbox.
- Slack facts beyond those the Decisions cite, from docs.slack.dev, read 2026-11-08:
  - An app keeps at most ten Socket Mode connections.
  - Each envelope needs an acknowledgement or Slack retries it, and the docs name no Socket Mode acknowledgement deadline.
  - Connections refresh every few hours, announced by a `disconnect` envelope.
  - `conversations.create` names allow lowercase letters, digits, `-` and `_`, up to 80 characters.
  - `conversations.replies` takes `oldest` under `channels:history`.

## Auto-mode assumptions

- The name's first segment is the hub's shortname, per the operator's `ly-neve-prime` example (D4).
- The prime's channel is never archived (D6), and the prime uses a channel, not the bot's direct message.
  One machine's bot has one DM shared by every hub there.
- Presence stays per hub, and `slack-presence-auto` is not joined (D9).
- Archiving happens when the beam is no longer listed and at `lym slack stop`, never at a relay crash (D6).
- Channels are made on first need (D5), and they are public.
- Thread replies are forwarded; files are named, not relayed (D7).
- The post icon is a fixed emoji per sender kind (D8).
- The user token is required on every machine (D3).

## Open risks

- Slack's docs do not say whether an archived channel's name stays taken.
  D5's step 2 handles both answers; the first live reopen confirms which one Slack gives.

## Acceptance criteria

- D1: with two hubs' relays on one machine, only the lock holder opens a connection.
  An event for a channel in hub B's map, read by hub A's holder, writes a hint into B's inbox, and B forwards the message Slack returns.
- D1: an envelope is acknowledged only after its hint write stood, and a failed write leaves it unacknowledged.
  An event no live hub maps is acknowledged and dropped.
- D1: a hint file holding text is never forwarded as text; a hint only triggers a read from Slack.
- D1: when the holder's relay ends, a waiting relay takes the lock and opens a connection.
- D1: a `disconnect` envelope, a read error or a pong missing 10 seconds after a ping opens a new connection.
  Reconnects back off from one second to sixty, and the wait resets only after a connection that lasted a minute.
- D2: a relay forwards an operator message under an injected clock with no wait beyond its hint's file event.
- D2: a lost hint for one message is covered by the read for the next, and a message both hinted and caught up is forwarded once.
- D2: a relay's start, and a resync hint after a reconnect, forward the top-level messages after each cursor.
  They also forward the replies to the last 24 hours' messages past each thread cursor.
- D2: concurrent map writes never move a cursor backwards.
- D3: `lym slack login` prompts naming the three kinds and files an `xoxb-`, `xapp-` or `xoxp-` token under its kind after Slack confirms it.
  It refuses any other prefix, naming the three, and keeps the bot token under the `token` key.
- D3: a machine missing any token is refused by the relay start before any spawn and by `RequireConfigured`, naming `lym slack login` and the missing kind.
- D3: only the unarchive and the bot's invite to a reopened channel use the user token.
  Neither is made for a channel another orch's entry in a live hub's map holds.
- D4: names come out as `ly-neve-prime` and `ly-neve-<slug>`, and a slug `prime` gives `ly-neve-prime_beam`.
- D4: a machine name with characters outside Slack's grammar is mapped onto it, and a name over 80 characters ends in the hash suffix at 80.
- D5: the prime's channel is ensured at relay start.
  A beam's is ensured at the first outbound mail routed to it, from its orch or a beam program, and every listed beam's when `lym slack away` sets presence `slack`.
  The operator is invited each time.
- D5: the map entry is written before the operator's invite, so a message written at once in a new channel is forwarded.
- D5: on `name_taken`, an archived channel of that name is reopened and a live one the bot belongs to is adopted.
  A channel another hub's or orch's entry holds is refused, with one notice to the hub's orch.
- D6: a beam no longer listed has its channel archived at the next wake, and a failed beams read archives nothing.
  `lym slack stop` ends the relay, then archives every open beam channel.
  The prime's channel is never archived.
- D6: an archived beam channel ensured again is reopened through the user token with the bot invited back (`already_in_channel` counts as done), and its cursors are kept.
- D6: mail from a sender whose beam is no longer listed posts to the prime's channel under the sender's name.
- D7: a message in a beam's channel reaches that beam's orch, and an `@<slug>` prefix is forwarded as plain text.
  No outbound post carries an answer hint.
- D7: `away`, `back` and `status` in a beam's channel set or read the hub's presence and are answered in that channel.
- D7: an operator thread reply and a `thread_broadcast` are forwarded to the channel's orch.
  A `file_share` message forwards its text and one `(file not relayed: <name>)` line per file, and other subtypes are skipped.
- D8: mail from a beam orch or beam program posts to that beam's channel, and mail from the hub's orch or a hub program to the prime's.
  Each carries the sender's name as `username` and its kind's `icon_emoji`, with no `[<sender>]` prefix.
- D8: the relay's dropped, stale and refused lines post to the message's channel.
- D8: a post refused with `is_archived` reopens the channel and posts again, and one refused with `channel_not_found` ensures the channel anew and posts again.
- D9: a presence change still notifies the hub's orch and every beam orch once.
- D10: `lym slack setup --operator <id>` writes the operator after `users.info` confirms it.
  A `setup.json` with an old channel key is read without error, and `status` lists the map's channels.
- Review: D10: `docs/slack-app.md` (new) carries a manifest with the app name, Socket Mode, the `message.channels` event, and every bot, user and app-level scope D10 lists.
- Review: D10: the `docs/overview.md` slack bullet and the `internal/slackengine` and `internal/slackcli` package documentation describe the push route, the channel per orch and the three tokens.
- D11: the Socket Mode client reads a `hello`, an `events_api` envelope and a `disconnect` from an `httptest` server, and acknowledges by `envelope_id`.

## PATTERN changes

- `PATTERN-agent-addressing` — changes: Naming or messaging an agent: names only via `internal/agentname`; messages only through creel; across hubs daemon to daemon with the sender confirmed, authority only for a trusted hub's orch.

## Notes for the plan writer

- The `PATTERN-agent-addressing` overview line keeps its text; its background's Slack boundary rule changes to:
  only `internal/slackengine` calls Slack, over the Web API and Socket Mode, reading its bot, app-level and user tokens only from `slack.json` in githubclient's credential directory.
  The user token serves only to unarchive a hub's own channel and invite the bot back.
  Operator text reaches an agent only as creel mail from `<shortname>:SLACK` of what Slack returned.
- The `PATTERN-daemon-wakeups` background lists the Slack relay's loops by name, and `cmd/lym/daemonwakeups_test.go` declares the unbounded-wait files.
  Both follow the loops this task leaves: the outbound read, the socket ping and reconnect, and the inbox watch.
- `slackengine.EnsureRelayWhileAway` and dobby's restart of a dead relay keep their logic; only the configured check they share changes (D3).
