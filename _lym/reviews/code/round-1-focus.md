---
round: 1
focus:
  - "Scope: boardengine/render.go, hubgeom/slackgeom.go and docs/overview.md change outside the Limits list; check whether the answer key licenses each."
  - "Per-orch button state in internal/slackcli/toggle.go and internal/slackengine/forwarding.go: persisted under .lym/slack/, survives restart, second press switches forwarding off and keeps the channel, a pair's orch gets no button."
  - "Relay lifecycle in internal/slackengine/relay.go, connection.go and start.go: does the relay run only while at least one orch is active, and does it stop when the last button goes off?"
  - "Turn-end hook in internal/slackcli/hook.go: does it post only for an orch whose button is on, with away still one machine-wide setting?"
  - "Channel naming in internal/slackengine/channelname.go: <machine>-<hub>-<project handle>-<project slug>, lower-cased, prime orch is the root project's p0-root, existing channels not renamed or archived."
  - "Removal of the [slack presence] notice: no mailing path is left in the relay, and role-slack.md drops the line."
  - "Error paths in inbound.go, keeper.go and the deleted switch.go: unhandled failures, leftover references, and races between button press and relay start or stop."
  - "Test economy: new or changed tests in slackengine, slackcli and reedengine against existing coverage; behaviours added without coverage."
---

The seed names where to look. It states no verdicts.
