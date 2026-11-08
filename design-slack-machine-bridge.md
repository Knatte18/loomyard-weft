# Let orchs on two machines talk through a Slack bridge channel

`slack-machine-bridge` · Note · feature, slack, creel, orch

## Seen (operator, 2026-11-08)

The operator wants the orchs on two machines (neve and OSL-1046) to be able to talk to each other through Slack, both machines connected at once.
Slack v2 (`slack-v2`, S1) keeps one app per machine and never lets two apps write to one channel, and it treats every inbound Slack message as the operator's.

## Idea

- A shared bridge channel, such as `ly-bridge`, that every machine's bot reads and writes.
- A message from another machine's bot is delivered as from that machine's orch, never as the operator, so one machine cannot command another in the operator's name.
- Built after Slack v2 lands; the operator wants it later, not now.
