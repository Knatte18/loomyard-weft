---
format: 5
approved: false
language: go
---

# Plan: slack-machine-bridge

Orchs on two machines, each with its own `lym-<machine>` Slack app in one workspace, exchange creel mail through a bridge channel per pair of orchs.
Creel journals an orch's post to `<machine>::<name>` and owns its hour; the Slack relay alone takes it, posts it with an envelope, and delivers the peer's envelopes back to creel through program-only verbs.
Mail from another machine is stamped `(other machine, …)`, and only an app the operator approved in Slack is read.
The cards run from the address parser through creel's journal, post, relay verbs and deliver verb, to the relay's records, commands, sending, receiving and upkeep.

## Card Index

1 — machine-address-parser — `internal/agentname` parses `<machine>::<name>` with the own-name refusal
2 — machine-mail-journal — creel's journal form, fold and retention carry for machine mail
3 — machine-outbound-post — creel journals an orch's post to another machine and refuses every other edge
4 — machine-relay-verbs — creel's program-only take, report and settle verbs and the machine mail hour
5 — machine-inbound-deliver — creel's deliver verb, dedupe, reply-to check, reply back and the machine stamp
6 — creel-bridge-wiring — the Go client's machine verbs and the bridge seam creel's daemon is told
7 — bridge-envelope-and-name — the envelope's sole formatter and parser and the bridge channel name
8 — bridge-records-and-calls — the peers record, bridge map, record paths and the Slack calls the bridge needs
9 — approve-and-revoke — the operator's `approve` and `revoke` in an orch channel, and the app id in `status`
10 — bridge-sending — the relay makes bridge channels and posts machine mail, with holds and the bridge poll
11 — bridge-receiving — the relay reads bridge channels, delivers to creel, and resyncs channels it lacks
12 — bridge-channel-upkeep — bridge channels take the new operator and are archived with their orch

## Shared Decisions

- `internal/agentname` imports the standard library alone (`TestAgentNameInvariant_StdlibOnly`), so `ParseMachineAddress` takes the machine rule as a function and every caller passes `machinename.CheckName`; D1's "must pass `machinename.CheckName`" holds through that argument.
- `internal/slackengine` imports `internal/creelengine`, so creel cannot call the Slack engine: creel learns this machine's names and whether an orch may send (button, tokens, setup, a button at all) through the `MachineBridge` seam, filled in `internal/creelcli` from `internal/slackengine`, as D3 states.
- The relay's program name has one constant, `creelengine.BridgeProgram`; `slackengine.SlackProgram` is defined as it, so creel's program check and the relay's claim cannot drift apart.
- D5's envelope has one formatter and one parser, in `internal/slackengine`. Creel's line check needs the envelope's width without importing the engine, so `creelengine.MachineEnvelopeWidth` bounds it from the envelope's fixed shape, and card 7's formatter test pins that the formatted line never exceeds that bound.
- The settle needs Slack's timestamp, which `PostAs` does not read, so `SlackClient` gains `PostTimestamped`; `PostAs` keeps reading nothing past `ok` for orch mail.
- The relay must tell an answered settle from a lost connection (D7: an accepted entry goes only once creel has answered), so creel exports `IsDaemonRefusal`.
- The machine mail hour is `outboundDeadline`, one hour from the post, as for mail to another hub; the relay's retention bound is `creelengine.MachineInboundRetention`, the journal's seven days, so the relay never skips within creel's dedupe window.
- The bridge map is keyed by channel id; the bridge-notice record's keys are `<kind>:<local orch> <peer>` on the send side and `<kind>:<channel id>` on the read side, so a mark outlives a map drop.

## PATTERN changes

- `PATTERN-agent-addressing` — changes: Naming or messaging an agent: names only via `internal/agentname`; only through creel; across hubs with the sender confirmed, across machines orch to orch via the Slack relay; authority only for a trusted hub's or approved machine's orch.
