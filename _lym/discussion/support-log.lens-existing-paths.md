```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code paths that already run against the state each decision introduces or changes.

**Findings**

- **MEDIUM — D3, existing start paths.** The relay start would accept a machine that `RequireConfigured` then refuses, so a relay could die young in a loop.
  - Evidence: `internal/slackengine/start.go`'s `starter.ensure` counts a machine as configured once `ReadToken` finds the bot token and `ReadSetup` finds the setup file.
  - D3 makes `RequireConfigured` refuse a machine missing the app-level or user token. `RunRelay` calls `RequireConfigured` at `relay.go:186`.
  - On a machine with only the old bot token, `EnsureRelay` and dobby's `EnsureRelayWhileAway` spawn a relay that refuses at once. Dobby counts each one as a start that died young, and gives up after five.
  - The Notes say `EnsureRelayWhileAway` is unchanged. The record is silent on making `ensure`'s check match D3's three tokens.
- **MEDIUM — D6, `lym slack stop`.** Archiving before the relay ends lets the still-live relay reopen what the stop just archived.
  - Evidence: `slackcli/wiring.go:70` wires stop to `slackengine.StopRelay`, which is `runproc.EndRole` on the relay role.
  - D6 has the CLI archive every open beam channel before it ends the relay. In that window, the live relay can get a beam orch's mail, be refused with `is_archived`, and reopen the channel (D8).
  - It can also hit a map write race on `channels.json` between the two processes.
  - The order should be: end the relay, then archive. Or the relay archives on its own shutdown.
- **MEDIUM — D5, presence set by the CLI.** The relay has no wake for a presence change made outside it, so D5's presence trigger can lag by up to a minute.
  - Evidence: `lym slack away` and `lym slack back` call `SetPresence` in the CLI process.
  - Today the relay sees the change through `presenceChanged` on each poll. D1 and D2 remove the poll, and the relay's only file wake is its inbox.
  - So D5's "ensure every live beam orch's channel when presence becomes `slack`" waits for the next outbound wake (up to 60 s), or for the next Slack event.
  - The record should name a wake on `presence.json`, such as an `internal/fswatch` watch on the relay's slack directory.
- **MEDIUM — D5 and D8, program mail.** D8 can route a beam program's mail to a beam channel that D5 never creates.
  - Evidence: dobby's wait reminders post to `<shortname>:SLACK` "signed as the notice is" (`beamcli/doc.go:428`, `dobbydeps.go:277`).
  - D8 routes `<shortname>:<slug>:<NAME>` to the beam's channel. D5 ensures a beam channel only at the beam *orch's* first mail, or when presence becomes `slack`.
  - A reminder for a beam whose orch has never posted therefore has no channel. The record does not say whether program mail ensures the channel or falls back to the prime's.
- **LOW — D10, existing readers of the setup's channel.** Code paths still read the setup's channel id.
  - `slackcli/cli.go`'s setup checks channel membership through `ConversationInfo`, and `status` prints `setup.ChannelID` at `cli.go:527`.
  - The outbound "dropped message" line (`outbound.go:197`) and its stale and refusal lines (`outbound.go:141`, `167`) post to `setup.ChannelID`.
  - D7 sets where inbound answers go, but no decision names which channel these relay lines post to. The sender's channel is the natural choice, and the prime's for a gone sender.
- **LOW — D3, token file rewrite.** This survives, but only if login keeps rewriting through `internal/state`.
  - `WriteToken` writes `tokenRecord{Token}` through `state.WriteJSON`, which keeps unknown keys. So filing an app or user token beside `token` survives a bot-token rewrite only while every writer goes through `internal/state`.
  - The record should keep the bot token under the existing `token` key, so a relay of an older binary on the same machine still reads it.

**Paths traced**

| Decision | Path | Result |
|---|---|---|
| D1 | holder ended by any `HubOwner` end route (`orchcli/cli.go:270`, tender, hibernate) | survives: the lock passes on |
| D2 | the old `cursor.json` | survives: ignored, cursors move to the channel map |
| D3 | `ReadToken` callers: `cli.go:275`, `start.go:172`, `RequireConfigured` | `start.go:172` breaks |
| D3 | `WriteToken` at `cli.go:226` | survives with the existing key kept |
| D5 | the presence trigger through the CLI's `SetPresence` | breaks |
| D5 | program mail through dobby's reminder | breaks, by a gap |
| D6 | `lym slack stop` through `StopRelay` | breaks, by ordering |
| D6 | relay crash and restart | survives: archived on the first wake |
| D6 | relay ended by another `HubOwner` route | survives: archived on the next start's wake |
| D8 | the existing `outboundText` and `programSlug` | survive |
| D10 | `cli.go:293` setup write | must change |
| D10 | `cli.go:527` status | misreads |
| D10 | `outbound.go:141`, `167`, `197` | misread |

**Tool calls**

1. Grepped for `ChannelID`, `RequireConfigured`, `ReadToken`, `StopRelay` and `WriteToken`, because my context did not have the readers of the changed state.
2. Grepped for `StopRelay` and `HubOwner`, and read `start.go`'s `ensure` and `cli.go`'s setup, because my context did not have the stop routes or the start's configured check.
3. Read `credentials.go` and grepped dobby's reminder sender, because my context did not have the token file shape or how the reminder is signed.
