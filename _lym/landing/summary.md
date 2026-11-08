# Slack relay: Socket Mode push, one channel per orch, two tokens

The per-hub Slack relay no longer polls one channel with `conversations.history`.
Each orch gets its own Slack channel, and an operator message wakes the relay by push within about a second.
Every hub on a machine shares that machine's one Slack app and bot.

## What changes

- **Push route.**
  One relay per machine, the holder of a machine socket lock beside `slack.json`, keeps a Socket Mode connection open.
  For each event it finds the live hub whose channel map holds the channel and writes a text-free wake hint into that hub's `.lym/slack/inbox/`.
  The hub's relay then reads Slack after the channel's cursor and forwards what Slack returns.
  A hint never carries message text, so a planted file can at most trigger a read.
- **One channel per orch.**
  Channels are named `<shortname>-<machine>-<orch>`, such as `ly-neve-prime`, and lym creates them on first need as public channels.
  The map in `.lym/slack/channels.json` holds each channel's id, cursors, thread floor and invited operator.
  Inbound mail is routed by channel; outbound mail by sender, posted under the sender's name with `:robot_face:` for an orch or `:gear:` for a program, with no `[<sender>]` prefix.
- **Threads and files.**
  Operator thread replies and `thread_broadcast` messages are forwarded.
  An attached file is never downloaded; the forwarded text names it as `(file not relayed: <name>)`.
- **Two tokens.**
  `lym slack login` asks for the bot token (`xoxb-`) and the app-level token (`xapp-`) in one run and files each by prefix.
  It writes both only when the app-level token's app id matches the bot token's, and it ends the live relays of the machine's hubs when it changes a stored token.
  No user token is stored or used.
- **Channel lifecycle.**
  The bot joins a channel itself with `conversations.join` after a `not_in_channel` refusal or a `name_taken` adoption.
  When a beam is no longer listed, its channel is renamed with a `-<UTC date>` suffix and archived; the prime's channel is never archived, and `lym slack stop` archives nothing.
  An archived channel is never reopened and lym never calls `conversations.unarchive`: an orch whose channel is archived gets a new channel under the freed name.
- **Setup and docs.**
  `lym slack setup --operator <user-id>` replaces `--channel`, and a running relay takes up the new operator with no restart.
  `lym slack status` lists the map's channels and reports `last_healthy`.
  `docs/slack-app.md` (new) carries the app manifest with every scope; `docs/overview.md`, the `slackengine` and `slackcli` package docs, `PATTERN-agent-addressing` and `PATTERN-daemon-wakeups` describe the new route.

## Design decisions

- One connection per machine, because Slack hands each payload to any one of an app's open connections.
  Other relays block on the lock and the first to get it connects once the holder ends.
  The relay is given the live-hubs list (from `hubregistry.Live` and `hubgeom.SlackGeometry`) and the connection as seams on `slackengine.RelayDeps`.
- Slack's history is the source of truth.
  A hint, a start or a reconnect triggers a read after the cursor, so lost, duplicated or reordered hints are harmless.
  A failed read stays owed in memory and is retried when the outbound read's wait ends.
- The holder pings every 30 seconds and reconnects on a missing pong, a `disconnect` envelope or a read error, backing off from one second to sixty.
  The file-event wake never starts within a second of the previous one, and a failed `fswatch.Watch` falls back to a backed-off check on disk.
- A channel is adopted on `name_taken` only when this machine's bot created it and no live hub's map gives it to another orch.
  An unreadable live hub's map, or a failed `hubregistry.Live` read, fails that test closed with a `[slack relay]` notice naming the file to move aside.
- The Socket Mode client is `github.com/coder/websocket` (new in `go.mod`), behind a seam so tests feed envelopes without a network.
- `EnsureRelay` and `EnsureRelayWhileAway` keep their signatures and answer `OutcomeNotConfigured` for a machine missing a token.
  The restart cap reads the new last-healthy stamp.

## To check by hand

- The tier 2 and tier 3 test suites were not run; only tiers 0 and 1 passed.
- Nothing was exercised against a live Slack workspace.
  Create the app from the `docs/slack-app.md` manifest, run `lym slack login`, and confirm that a message in a new orch channel reaches the orch.
  Also confirm that a beam's channel is renamed and archived when the beam goes, and that a second hub on the machine is woken only for its own channels.
- The `hello` message's `connection_info.app_id`, which login reads to match the app-level token to the bot token, is documented only by example.
  Confirm it against a real connection.
- Slack's docs name no acknowledgement deadline or keepalive; the ten-second hint-write limit and the 30-second ping are this change's own choices.
- Plan-review finding, in the `internal/slackcli` dialer seam: the shape of the new dialer seam may not match what the login card's tests expect.
  Check the seam's signature against its callers.
- Plan-review finding, in the live-hubs error path: the advice for a failed `hubregistry.Live` read may contradict the hubs-record advice elsewhere.
  Check that the notices and refusals name the same way forward.
