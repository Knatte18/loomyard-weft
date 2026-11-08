---
format: 5
approved: false
language: go
---

# Plan: Slack relay with one push-woken channel per orch

Replace the per-hub Slack relay's single polled channel with one channel per orch, woken by Socket Mode push over one connection per machine.
The cards build the client calls and the Socket Mode client first, then the channel names and map, the two-token login and the operator-only setup,
then the relay's engine pieces (the socket holder, the channel keeper, the inbound and outbound sides), and last the relay loop that wires them together with its docs.
The decision record's operator-added entry governs where it differs from D3, D5, D6, D8 and D10: no user token, the bot joins by itself, and a retired channel is renamed with a date and archived, never reopened.

## Card Index

1 — api-web-methods — add every Web API call the push route makes to `API`
2 — socket-mode-client — add the Socket Mode client over `github.com/coder/websocket` behind a `Socket` seam
3 — channel-names — name each orch's channel and a retired channel's dated name
4 — channel-map-geometry — add the hub's channel map with forward-only cursors and the push route's paths
5 — two-token-login — store the bot and app-level tokens and log both in through one run
6 — setup-operator-status — reduce setup to the operator and list the map's channels in status
7 — slack-client-surface — widen the relay's Slack seam and stamp the last-healthy time per outbound read
8 — socket-holder — hold the machine's one Socket Mode connection and write wake hints into each hub's inbox
9 — channel-keeper — make, adopt, join, invite and retire the hub's channels
10 — inbound-push — read each channel after its cursor on a hint, a resync or the start, routed by channel
11 — outbound-routing — post each message to its sender's channel under the sender's name and icon
12 — relay-push-loop — run the relay on push with its wakes, retries and docs

## Shared Decisions

- `github.com/coder/websocket` is required at `v1.8.15`, the newest version the Go proxy lists on 2026-11-08 (`go list -m -versions`); it needs no further module.
- `slack.json` keeps the bot token under `token` and adds `app_token` and `app_id`; `setup.json` keeps an older binary's `channel_id`, read only by the relay's missing-token line.
- The relay state keeps its record key `last_slack_ok`; its meaning becomes the last-healthy time D1 stamps, and `lym slack status` prints it as `last_healthy`.
- The hubs record's directory is `standalonestate.UserStateDir()`, as creel's own reads of `hubregistry.Live` use.
- The channel map is keyed by the orch's creel name, `<shortname>:orch` for the prime and `<shortname>:<slug>:orch` for a beam.
- The relay builds one channel keeper, which its inbound, its outbound and its wake loop share.
- The Socket Mode dial reads the `hello` within `HelloTimeout`, ten seconds, a parameter of `NewDialer` so a test passes a short bound.
- Pattern-Departure: `PATTERN-lym-files` puts temp files under `lymtemp`, while card 8's hint write creates its temp file in the hub's `.lym/slack/`, outside the inbox.
  D1 names that placement, and it keeps the rename into the inbox on one filesystem, so a hint appears whole or not at all.

## PATTERN changes

- `PATTERN-agent-addressing` — changes: Naming or messaging an agent: names only via `internal/agentname`; messages only through creel; across hubs daemon to daemon with the sender confirmed, authority only for a trusted hub's orch.
