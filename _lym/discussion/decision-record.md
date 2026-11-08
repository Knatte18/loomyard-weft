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

The relay is told, as a seam like its `SlackClient`, how to list the machine's live hubs with each hub's Slack geometry.
`internal/slackcli` fills that seam from `hubregistry.Live` and `hubgeom.SlackGeometry`.

The holder opens the connection with `apps.connections.open` using the app-level token.
For each `events_api` envelope it finds the hub whose channel map (D5) holds the event's channel, reading every live hub's map.
It writes a wake hint into that hub's inbox directory, `.lym/slack/inbox/`, then acknowledges the envelope.
A hint is a file named `<channel id>` for a channel, or `<channel id>_<thread ts>` for a thread reply.
A later hint for the same channel or thread overwrites it.
The holder writes a hint by creating a temp file in the hub's `.lym/slack/` directory, outside the inbox, then renaming it into the inbox.
For a `thread_broadcast` the holder writes the channel hint, since the top-level read forwards it (D2).
The hint carries no message text.
An event whose channel no live hub maps is acknowledged and dropped with a Debug log line, while every live map reads.
While any live map cannot be read or decoded, an event no readable map holds is left unacknowledged for Slack's retry;
while the `hubregistry.Live` read fails, every event is.
A holder whose `hubregistry.Live` read fails mails its own hub's orch one `[slack relay]` notice per episode naming the hubs record's path,
and first posts the same line to its prime's channel when its own map reads and holds that channel.
Its way forward is D3's: moving that record aside, after which `lym creel inbox` in each hub records it again.
While the episode lasts, the holder reads `hubregistry.Live` again at each ping, so the repair takes effect with no new event;
the read that stands ends the episode and owes every hub it names a resync hint.

One envelope's map reads and hint write have ten seconds.
When the hint write fails or passes that limit, the holder acknowledges the envelope and marks that hub as owed a resync hint.
A resync hint that fails after a connection opens is owed the same way.
The holder also owes a resync hint to each hub a `hubregistry.Live` read names that its previous read did not,
and to every hub it names after a read that failed.
At each ping the holder writes every owed `resync` hint (D2) until the write stands, and the resync read covers the lost hint.
The hub's orch gets one `[slack relay]` notice per episode naming the inbox directory and the write's error,
and the other hubs' events go on as before.

The holder sends a WebSocket ping every 30 seconds and reconnects when no pong comes within 10 seconds.
It also reconnects on a `disconnect` envelope or a read error.
Reconnects back off from one second, doubling to sixty.
The wait returns to one second only after a connection that lasted a minute.
A connection refused or disabled by Slack mails the holder hub's orch one `[slack relay]` notice per episode, naming `lym slack login`.
Before a re-exec onto a replaced binary (D2), a holder closes its connection and releases the machine socket lock;
after the exec it waits its turn for the lock again.
A holder whose re-exec fails rejoins the machine socket lock wait, as it does after an exec.

The relay stamps the relay state's last-healthy time, the field `LastSlackOK` holds today,
at each Socket Mode pong while it holds the connection, and at the end of each outbound read for every relay.
The restart cap's lifetime reads that stamp, and `lym slack status` reports it as `last_healthy` in place of `last_slack_ok`.

Rationale:
Slack hands each payload to any one of an app's open connections, and says to assume no pattern of distribution
(docs.slack.dev, "Using Socket Mode", read 2026-11-08: "each payload may be sent to any of the connections").
So one connection per machine is the only way one hub never gets another's events.
The OS lock hands it to the next relay when its holder ends.
A hint without text, checked against the relay's own map (D2),
means a planted file can at most make a relay read one of its own channels from Slack.
It can never put words in the operator's mouth.
The ten-second limit bounds a holder stalled on a map lock or a hint write, which the ping does not see;
Slack's docs name no acknowledgement deadline.
An owed resync keeps one hub's broken inbox from stalling the other hubs' push.
The ping catches a half-open connection after a suspend or a network change.
Slack's docs name no keepalive of their own (same page).

Bound:
The route wakes only the hub whose map holds the event's channel, and the woken relay forwards only what Slack returns for that channel (D2).
The ping is a bounded 30-second wake that spawns nothing.

### D2 — Slack's history is the source; a hint, a start or a reconnect triggers a read after the cursor

Decision:
Each channel's map entry carries a cursor, the newest top-level timestamp handled.
It also carries a thread cursor per thread, the newest reply handled, and a thread floor.
A thread cursor is pruned once its thread has had no reply for 24 hours,
and the prune raises the thread floor to 24 hours before the prune.
An adoption, or a reopen with no map entry (D5), sets the thread floor to the channel's newest message.
A thread with no cursor starts at the later of its parent's timestamp and the channel's thread floor,
so no reply the channel already handled is forwarded again.

A relay reads a channel with `conversations.history` after its cursor at each of these:

- a hint for that channel;
- a resync hint, for each open channel;
- the relay's start, for each open channel.

A thread hint reads `conversations.replies` for that thread after its thread cursor, or after its start value when it has none.
A thread read skips the message whose timestamp equals its thread timestamp, since the top-level read handles it.
A resync or start read also takes every top-level message of the last 24 hours whose `latest_reply` is past its thread cursor,
or past its start value when it has none, and reads those replies after it.
A `thread_broadcast` is forwarded only by the top-level read;
a thread read skips it and moves the thread cursor past it.
After each connection it opens, the holder writes one `resync` hint into every live hub's inbox, overwriting any left there.
Besides `resync`, a relay acts on a hint only when its name parses as a channel id, or a channel id and a thread timestamp,
and the channel is in the relay's own map.
It removes any other hint unread, with a Debug log line.
The drain ignores names starting with a dot.
Cursors only move forward, and every map write is a read-modify-write under the map's `internal/state` lock.
No Slack call is made while a map lock is held.

One goroutine drains the relay's whole inbox at every wake, so two wakes never read one channel at once.
The relay removes each hint before it makes the read that hint triggers,
so a hint written during the read is a new file and a new wake.
A wake is a file event or the outbound read's wait ending.
A read a hint, a resync hint or the start triggered that fails on a transport error, a server error or a rate limit
stays owed, by its channel or thread, in an in-memory set; an owed start or resync read keeps its 24-hour thread scan.
At each wake from the outbound read's wait the relay makes every owed read again until it stands,
and one owed after a rate limit waits out that limit's retry wait first.
A failed owed read counts as finding nothing for the outbound read's wait, so with no outbound mail the retries back off to sixty seconds.
A relay that ends loses the set, and its next start reads every open channel.
The relay creates the inbox directory, which holds only hints, before it watches it,
and it watches the Slack directory for the presence file alone, through `internal/fswatch`'s names filter.
A file-event wake never starts within a second of the previous wake's start;
an event inside that second wakes the relay at its end.
When `fswatch.Watch` fails at open, the relay checks the inbox and the presence file on disk instead,
from one second doubling to sixty while it finds nothing, and back to one second after it finds a hint.
After each outbound read the relay checks whether the lym binary was replaced and re-execs onto it.

Rationale:
Reading from Slack after the cursor makes order, retries and duplicates harmless.
For example, a hint lost for one message is covered by the read for the next, and a message both hinted and caught up is forwarded once.
The removal stays before the read, since it orders the read after any hint written meanwhile;
the owed set, not the hint, carries a read that failed, so an operator who sends nothing more still gets the message forwarded.
The watch sees none of the relay's records or its log, so a Slack call's stamp or a failure's log line never wakes it.
The relay's own hint removals and presence writes do wake it, under the one-second floor.
The gaps push leaves are a relay not running, the lock hand-over and a reconnect.
A start or a new connection follows each of them, and its catch-up runs then.
A hint lost to a failed or slow hint write is covered by the resync hint the holder owes that hub (D1).
Bound: a thread reply sent during a gap to a message over 24 hours old is not recovered,
and neither is a reply no read caught before a prune raised the thread floor past it.

### D3 — Three tokens in the machine credential file, filed by their prefix

Decision:
`slack.json` keeps the bot token (`xoxb-`) under its existing `token` key and adds the app-level token (`xapp-`) and the operator's user token (`xoxp-`).
`lym slack login` keeps the token read `slack-login-prompt` landed on `main` as `fd159ff2`:
one line with echo off on a terminal, and piped input read to its end.
Its `tokenPrompt` and its empty-token refusals are reworded to name the three kinds.
It files the token by its prefix, and refuses any other prefix, naming the three it takes.
It confirms a bot or user token with `auth.test`.
An app-level or user token logged in with no bot token stored is refused, naming `lym slack login` with the bot token first.
It confirms an app-level token by opening a Socket Mode connection, reading the `hello` message's `connection_info.app_id`,
and closing the connection.
An event Slack hands that connection in the meantime goes unacknowledged, so Slack retries it.
Login stores that app id in `slack.json` beside the app-level token, so a later bot-token login matches against it.

Login writes a token only when it matches the tokens already stored.
The bot and user tokens must give one `team_id` from `auth.test`.
The app-level token's app id must be the bot token's, which `bots.info` gives for the `bot_id` from `auth.test`.
An app-level or user token that does not match the stored bot token is refused,
naming the bot token's kind and `lym slack login` with a token of that app as the way forward.
A bot token that does not match the stored tokens replaces the stored bot token and removes each stored token it does not match,
naming each removed kind for the operator to log in again.
A login that changes a stored token ends the relay of each hub `hubregistry.Live` lists, through its relay record as `lym slack stop` ends its own,
and names each hub whose relay it ended, with `lym slack start` as the way forward.

`slackengine.RequireConfigured` refuses a machine missing any of the three tokens, or a hub missing its setup file,
naming the missing kind and the verb that writes it, `lym slack login` or `lym slack setup`.
`lym slack start` and `lym slack away` refuse through it.
The relay makes the same check only after it takes its relay lock and makes its first creel read.
`EnsureRelay` and `EnsureRelayWhileAway` answer `OutcomeNotConfigured` with no error for the same machine or hub,
so orch bring-up and the anchor change neither warn nor fail, and dobby's restart never spawns a relay that refuses at once.
A relay that finds a token missing at that check, as after a re-exec onto a binary of this task,
mails the hub's orch one `[slack relay]` notice naming the missing kind and `lym slack login`, then exits.
When it has the bot token and the map holds the prime's channel, it first posts the same line there.
When the map does not hold the prime's channel and `setup.json` still carries an older binary's channel key,
it first posts the same line to that channel with the bot token, the only read of that key.

The user token is used only for `conversations.unarchive` and for inviting the bot into a channel:
one it reopened (D6), one D5 adopts on `name_taken` that the bot is not a member of,
or one where a post, a start read or a resync read was refused with `not_in_channel` (D8).
Each call is made only for a channel held by no other orch's entry in any live hub's map, and either:

- in this hub's map for one of its orchs;
- or, on D5's `name_taken` path, named this hub's own D4 name for one of its orchs, with this machine's bot user as its creator.

A live hub's map that cannot be read or decoded fails that test closed:
no unarchive, invite or adopt is made, and the ensuring hub's orch gets one `[slack relay]` notice naming the map's path.
Its way forward is moving that file aside, after which that hub's relay ensures its channels anew (D5).
A `hubregistry.Live` read that fails fails the test closed the same way, with one notice naming the hubs record's path.
Its way forward is moving that record aside; `lym creel inbox` in each hub, a minute on, records it again.
No token is ever config, an environment variable or a log line.

Rationale:
Socket Mode needs an app-level token (docs.slack.dev, `apps.connections.open`).
Slack refuses a bot token for `conversations.unarchive`
(docs.slack.dev, `conversations.unarchive`, read 2026-11-08: "Bot tokens (xoxb-...) cannot currently be used to unarchive conversations").
The operator asked to reopen an archived channel with its history.
The prefixes tell the kinds apart, so login needs no flag.
Keeping the `token` key lets an older binary on the machine still read the bot token.
A login is operator input at a trust boundary.
An app-level token of another app would make this holder one of that app's connections, handed events no map here holds.
The bot token anchors the match, so a machine moving to a new app logs in its bot token first and the other two after.
D4 names can repeat across machines, so a channel outside the map needs the creator test, not the name alone,
to keep this machine's user token off another machine's channel.
A channel in the map was made or adopted by this hub, so it still passes after a move to a new app.
The invite after a post, a start read or a resync read refused with `not_in_channel` brings the new bot back in.
Bound: the user token can act in any public channel the operator can reach, and only the calls above, on the channels above, use it.
The map test sees only the hubs the live-hub list names:
a hub absent from it, after its record is moved aside or before its creel daemon first records it, is unseen until it records itself.

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
The machine segment keeps each machine's channels apart in the sidebar, but a name can repeat across machines:
`machinename.FromHostname` keeps the part before the first dot, and `-` is legal inside a segment.
The sort keeps each hub's channels together, and each machine's apart under it.
A machine's hubs can share a shortname: `hubregistry.Live` lists them all, and only cross-hub mail refuses them.
So D5's creator test, not the name, keeps another machine's app off a channel, and D5's map test keeps two hubs off one.

### D5 — The channel map and when a channel is made

Decision:
Each hub keeps `.lym/slack/channels.json`, written through `internal/state`.
It maps each orch name to its channel's id, name, cursors, thread floor, the operator it invited and whether lym archived it.
A beam counts as listed when a successful `slackengine.Hub` read names it.

The relay ensures:

- the prime's channel at its start;
- a beam's channel at the first outbound mail routed to it, from its orch or a program (D8);
- every listed beam's channel at the relay's start when presence is `slack`,
  and at each wake that reads presence `slack` after it read `terminal` (D2);
- every open channel whose entry records another operator than `setup.json`'s, or none, at each wake from the outbound read's wait.

One relay ensures one orch's channel at a time, under a per-orch lock in the relay.
An ensure that, once it holds that lock, finds the map now holds the channel treats it as one the map holds.
Each ensure of a channel the map holds invites the setup's operator when the entry records another operator or none,
until the invite stands.
A channel stands once its map entry is written, so a refused invite waits for the list's last trigger.
The relay reads `setup.json` at every wake, and no watch covers that file (D2).
So `lym slack setup --operator <user-id>` reaches the inbound filter (D7) at the next wake,
and the invites at the next wake from the outbound read's wait, with no relay restart.
A refused invite counts as finding nothing for that wait, as a failed owed read does (D2),
so with no outbound mail the retries back off to sixty seconds.
An invite Slack refuses mails the hub's orch one `[slack relay]` notice per episode,
naming the refusal's code and `lym slack setup --operator <user-id>`.

Ensuring a channel the map does not hold:

1. `conversations.create` for the D4 name as a public channel, then the map entry at once, with an empty cursor so the first read starts at the channel's beginning.
2. On `name_taken`, `conversations.list` with archived channels included, to find the channel of that name.
   - When its `creator` is this machine's bot user, the `user_id` `auth.test` gives for the bot token,
     and no live hub's map gives that channel to another orch:
     - an archived one is reopened (D6), and the reopen writes a new entry with its cursor and thread floor at the channel's newest message, as adoption does;
     - a live one is adopted, with its cursor and thread floor at its newest message;
       when the bot is not a member, it is first invited with the user token under D3's bound.
   - Any other is refused, and the hub's orch gets one `[slack relay]` notice naming the channel.
     When another hub's map holds the channel, the notice names the way forward of `hubregistry`'s duplicate-shortname refusal:
     remove the extra clone of the hub, since a recorded shortname never changes.
     When another bot created the channel, the notice names renaming that channel in Slack, which frees the name.
   - A live hub's map that cannot be read or decoded, or a failed `hubregistry.Live` read, refuses the same way (D3).
3. `conversations.invite` of the operator, where `already_in_channel` counts as done, recorded in the entry once it stands.

A refused prime's channel is ensured again at each wake from the outbound read's wait until it stands,
and a refused beam's channel at its next routed mail.

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
An archive Slack refused is retried only at a wake from the outbound read's wait, which backs off, never at a file event.
`lym slack stop` ends the relay first, then archives every open beam channel of the hub from the CLI.
lym never archives the prime's channel.
A relay that dies and restarts archives nothing on its own account.

An archived entry is reopened when D5 would ensure its channel.
To reopen, the relay calls `conversations.unarchive` with the user token.
It then invites the bot with the user token, where `already_in_channel` counts as done, and marks the entry open.
When the map already holds the entry, the cursors stay where it left them.
Mail whose sender is a beam orch or program no longer listed posts to the prime's channel, under the sender's name.
While the beams read fails, mail from a beam whose channel the map holds open posts there, and other beam mail waits for the next read.

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
The relay's dropped and refused lines about a message post to that message's channel.
Its stale line, one count of the hub's stale mail, posts to the prime's channel.
A post refused with `is_archived` reopens the channel (D6) and posts again.
A post refused with `not_in_channel` invites the bot back with the user token under D3's bound and posts again.
A start or resync read refused with `not_in_channel` invites the bot back under the same bound and reads again.
A post refused with `channel_not_found` drops the map entry, ensures the channel anew (D5) and posts again.
Each refused post counts once toward the existing per-message cap of five Slack refusals, `refusalCap` in `internal/slackengine/outbound.go`.
A reopen, invite or ensure after it that Slack refuses adds no second count.
One that fails on a transport error, a server error or a rate limit counts toward no cap, as a post's does today.

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
A running relay takes up the new operator with no restart (D5).
The `--channel` flag and the channel membership check go.
A `setup.json` written by an older binary keeps its channel key, which only D3's missing-token line reads.
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

### Added after Discussion (arbiter, 2026-11-08): Round-3 review findings answered by the round-3 fix

Decision: The three findings open at the round-3 hard cap stand answered by the record as fixed, and need no further edit: failed-hinted-read-never-retried by D2's in-memory owed-read set, made again at each wake from the outbound read's wait, with its acceptance row; operator-invite-never-retried by D5's last ensure trigger at each wake from the outbound read's wait, with setup.json read at every wake and D10's no-restart line, with its acceptance rows; live-hub-list-read-unbounded by D1's per-episode notice to the holder hub's orch and prime channel, with the list read again at each ping, with its acceptance row.

Rationale: Each fix reuses an existing wake (the outbound read's wait or the holder's ping), so no loop is added and scope does not widen; the round-3 fixer report and the record's D1, D2, D5, D10, Constraints and acceptance criteria show each in place.

## Constraints

- `PATTERN-agent-addressing`: the overview line keeps its text; its background's Slack boundary rule changes to:
  only `internal/slackengine` calls Slack, over the Web API and Socket Mode, reading its bot, app-level and user tokens only from `slack.json` in githubclient's credential directory.
  The user token serves only to unarchive a hub's own channel and invite the bot into one.
  Operator text reaches an agent only as creel mail from `<shortname>:SLACK` of what Slack returned.
- `PATTERN-child-processes`: no new process kind; the relay keeps its double-fork start and its `runproc.HubOwner` record.
- `PATTERN-daemon-wakeups`: the inbound poll goes.
  The inbox and presence wake is an `internal/fswatch` event on the inbox directory and the presence file alone,
  and a file-event wake never starts within a second of the previous wake's start.
  When the watch fails at open, a check of those files on disk backs off from one second to sixty.
  The lock wait blocks without waking.
  The 30-second ping spawns nothing, and the reconnect backs off from one second to sixty.
  The outbound read keeps its wait; an archive Slack refused, an owed channel read and a held channel's operator invite wait for it,
  so no new loop is added.
- `PATTERN-lym-files`: the hub's new files sit under its `.lym/slack/`, with paths built by `hubgeom.SlackGeometry`.
  The relay reaches other hubs' maps and inboxes only through the live-hubs seam `internal/slackcli` fills from
  `hubregistry.Live` and `hubgeom.SlackGeometry` (D1).
  `lym slack login` reaches other hubs' relay records through the same `hubregistry.Live` list.
  The machine socket lock sits beside `slack.json` in the credential directory, which is machine state outside every hub.
- `PATTERN-tolerant-records`: the channel map and the token file are written through `internal/state`.
- `PATTERN-config-layers`: no config key is added; tokens, operator and channels are never config.
- `PATTERN-refusal-way-forward`: each new refusal of `login`, `setup`, the relay start and `RequireConfigured` names its way forward.
  Every existing refusal or notice whose way forward names `lym slack setup --channel` is reworded to `lym slack setup --operator <user-id>`.
  Every way forward that lists bot scopes names the D10 manifest's scopes and `docs/slack-app.md` (new).
  The relay's refused-read notice names `lym slack login` and the manifest, not `lym slack setup`.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: the relay tests stay tier 0, with an injected clock and fakes for the Slack client, the connection and the mailbox.
- Slack facts beyond those the Decisions cite, from docs.slack.dev, read 2026-11-08:
  - Each envelope needs an acknowledgement or Slack retries it, and the docs name no Socket Mode acknowledgement deadline.
  - Connections refresh every few hours, announced by a `disconnect` envelope.
  - `conversations.replies` takes `oldest` under `channels:history`.
  - `auth.test` answers `team_id` and `user_id`, and `bot_id` for a bot token, and names no app id (`auth.test`).
  - `bots.info` answers the bot's `app_id` under `users:read` (`bots.info`).
  - The `hello` example of "Using Socket Mode" carries `connection_info.app_id`; no prose there describes the field.
  - A conversation's `creator` is the id of the member who created it (the conversation object page).

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

None.

## Acceptance criteria

- D1: with two hubs' relays on one machine, only the lock holder opens a connection.
  An event for a channel in hub B's map, read by hub A's holder, writes a hint into B's inbox, and B forwards the message Slack returns.
- D1: an envelope is acknowledged only after its hint write stood.
  A failed hint write, or one past the ten-second limit, is acknowledged and owes that hub a resync hint,
  written at each ping until it stands, while the other hubs' events go on.
  An event no live hub maps is acknowledged and dropped while every map reads, and left unacknowledged while one cannot be read.
  Every event is left unacknowledged while the `hubregistry.Live` read fails,
  and once it reads again each listed hub gets a resync hint, as does a hub newly listed.
- D1: a failing `hubregistry.Live` read mails the holder hub's orch one notice naming the hubs record's path,
  posted first to the prime's channel when the holder's map holds it.
  The holder reads the list again at each ping, so a read that stands after the repair ends the episode with no new event.
- D1: a hint file holding text is never forwarded as text; a hint only triggers a read from Slack.
- D1: when the holder's relay ends, a waiting relay takes the lock and opens a connection.
- D1: a `disconnect` envelope, a read error or a pong missing 10 seconds after a ping opens a new connection.
  Reconnects back off from one second to sixty, and the wait resets only after a connection that lasted a minute.
- D1: a connection Slack refuses or disables mails the holder hub's orch one `[slack relay]` notice per episode naming `lym slack login`.
- D1: a relay idle for over ten minutes, then ended, does not count as a young death toward the restart cap.
- D2: a hint written into the inbox directory wakes the relay through its file event alone,
  and the relay forwards the operator message under an injected clock with no further wait.
- D2: the relay's writes to its relay state, outbox and channel map and its log wake nothing.
  An event inside a second of the previous wake's start wakes the relay at that second's end.
- D2: a hint naming a channel outside the relay's map reads nothing from Slack and is removed.
- D2: a hint written while its channel's read is in flight is read at the next wake.
- D2: a hinted read that fails on a transport error, a server error or a rate limit is made again when the outbound read's wait ends,
  with no new hint, and forwards the operator message once it stands.
  A failed start or resync read is made again the same way, its thread scan included.
- D2: a lost hint for one message is covered by the read for the next, and a message both hinted and caught up is forwarded once.
- D2: a relay's start, and a resync hint after a reconnect, forward the top-level messages after each cursor.
  They also forward the replies to the last 24 hours' messages past each thread cursor.
- D2: a thread cursor with no reply for 24 hours is pruned from the map, and the thread floor rises to 24 hours before the prune.
  A later reply to that thread is forwarded, and no earlier reply in it is.
- D2: a channel adopted on `name_taken` forwards no reply written before the adoption at the next start.
- D2: concurrent map writes never move a cursor backwards.
- D3: `lym slack login` prompts naming the three kinds and files an `xoxb-`, `xapp-` or `xoxp-` token under its kind after Slack confirms it.
  It refuses any other prefix, naming the three, and keeps the bot token under the `token` key.
- D3: login refuses a user token of another team or an app-level token of another app than the stored bot token's.
  A bot token of another app replaces the stored one and removes the tokens it does not match, naming them.
- D3: a machine missing any token is refused before any spawn by `lym slack start`, `lym slack away` and `RequireConfigured`,
  naming `lym slack login` and the missing kind.
  `EnsureRelay` answers `OutcomeNotConfigured` with no error for it.
- D3: a relay started with only a bot token takes its lock and makes its first creel read,
  then mails the hub's orch one notice naming the missing kinds and `lym slack login`,
  posts the same line to the prime's channel when the map holds it, and exits.
  A relay re-exec'd from an older binary's state, with no map, posts the line to the old setup channel.
- D3: only the unarchive and the bot's invite to a reopened channel, one adopted without the bot,
  or one whose post, start read or resync read was refused with `not_in_channel` use the user token.
  Neither is made for a channel another orch's entry in a live hub's map holds,
  or while a live hub's map or the live-hub list cannot be read.
- D3: an app-level or user token logged in with no bot token stored is refused, naming the bot token first,
  and a bot-token login after an app-level token matches against the app id login stored.
- D3: a login that changes a stored token ends every live relay of the machine's live hubs and names each such hub with `lym slack start`.
- D4: names come out as `ly-neve-prime` and `ly-neve-<slug>`, and a slug `prime` gives `ly-neve-prime_beam`.
- D4: a machine name with characters outside Slack's grammar is mapped onto it, and a name over 80 characters ends in the hash suffix at 80.
- D5: the prime's channel is ensured at relay start.
  A beam's is ensured at the first outbound mail routed to it, from its orch or a beam program.
  Every listed beam's is ensured by a relay `lym slack away` starts, by a relay restarted while presence is already `slack`,
  and at a wake that reads presence `slack` after `terminal`.
  An operator invite that failed is retried when the outbound read's wait ends.
  A changed operator is invited into every open channel at the next such wake, with the relay still running.
  An invite Slack refuses mails the hub's orch one notice per episode naming `lym slack setup --operator <user-id>`.
  Two ensures of one channel in one relay run one after the other, and the second makes no Slack call to create or adopt.
- D5: the map entry is written before the operator's invite, so a message written at once in a new channel is forwarded.
- D5: on `name_taken`, an archived channel of that name is reopened and a live one is adopted, the bot invited first when it is not a member.
  A reopen on `name_taken` with no map entry forwards no message written before the reopen.
  A channel another hub's or orch's entry holds is refused, with one notice to the hub's orch.
  An archived channel of the D4 name that another bot created is not unarchived.
  A refused prime's channel is ensured again when the outbound read's wait ends.
- D6: a beam no longer listed has its channel archived at the next wake, and a failed beams read archives nothing.
  An archive Slack refused is retried when the outbound read's wait ends, never at a file event.
  `lym slack stop` ends the relay, then archives every open beam channel.
  The prime's channel is never archived.
- D6: an archived beam channel ensured again is reopened through the user token with the bot invited back (`already_in_channel` counts as done), and the cursors of an entry the map held are kept.
- D6: mail from a sender whose beam is no longer listed posts to the prime's channel under the sender's name.
  While the beams read fails, a beam's mail posts to its open channel, and mail of a beam with no open channel waits.
- D7: a message in a beam's channel reaches that beam's orch, and an `@<slug>` prefix is forwarded as plain text.
  No outbound post carries an answer hint.
- D7: `away`, `back` and `status` in a beam's channel set or read the hub's presence and are answered in that channel.
- D7: an operator thread reply and a `thread_broadcast` are forwarded to the channel's orch.
  A `thread_broadcast` is forwarded once, whether its channel hint or a start read that takes its channel and its thread finds it.
  A `file_share` message forwards its text and one `(file not relayed: <name>)` line per file, and other subtypes are skipped.
- D8: mail from a beam orch or beam program posts to that beam's channel, and mail from the hub's orch or a hub program to the prime's.
  Each carries the sender's name as `username` and its kind's `icon_emoji`, with no `[<sender>]` prefix.
- D8: the relay's dropped and refused lines post to the message's channel, and its stale line to the prime's channel.
- D8: a post refused with `is_archived` reopens the channel and posts again, one refused with `not_in_channel` invites the bot back and posts again,
  and one refused with `channel_not_found` ensures the channel anew and posts again.
  A start or resync read refused with `not_in_channel` invites the bot back and reads again.
- D9: a presence change still notifies the hub's orch and every beam orch once.
- D10: `lym slack setup --operator <id>` writes the operator after `users.info` confirms it.
  A `setup.json` with an old channel key is read without error, and `status` lists the map's channels.
- Review: D10: `docs/slack-app.md` (new) carries a manifest with the app name, Socket Mode, the `message.channels` event, and every bot, user and app-level scope D10 lists.
- Review: D10: the `docs/overview.md` slack bullet and the `internal/slackengine` and `internal/slackcli` package documentation describe the push route, the channel per orch and the three tokens.
- D11: the Socket Mode client reads a `hello`, an `events_api` envelope and a `disconnect` from an `httptest` server, and acknowledges by `envelope_id`.

## PATTERN changes

- `PATTERN-agent-addressing` — changes: Naming or messaging an agent: names only via `internal/agentname`; messages only through creel; across hubs daemon to daemon with the sender confirmed, authority only for a trusted hub's orch.

## Notes for the plan writer

- The `PATTERN-daemon-wakeups` background lists the Slack relay's loops by name, and `cmd/lym/daemonwakeups_test.go` declares the unbounded-wait files.
  Both follow the loops this task leaves: the outbound read with the binary check after it, the socket ping and reconnect,
  and the inbox watch with its on-disk fallback.
- `slackengine.EnsureRelayWhileAway`'s restart cap keeps its logic;
  its lifetime reads the last-healthy time D1 stamps, in place of the last Web API call that succeeded.
