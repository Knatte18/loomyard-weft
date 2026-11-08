# Let orchs on two machines talk through a Slack bridge channel

`slack-machine-bridge` · Note · feature, slack, creel, orch

## Seen (operator, 2026-11-08)

The operator wants the orchs on two machines (neve and OSL-1046) to be able to talk to each other through Slack, both machines connected at once.
The Slack relay rebuild (`slack-push-channels`, built from the Sketch `slack-v2`) keeps one app per machine, never lets two apps write to one channel, and treats every inbound Slack message as the operator's.
Creel today carries mail between the hubs of one machine only: a name `<hub>:<strand>` resolves through the machine record, and the receiving daemon confirms the sender by asking the sending daemon, which cannot work across machines.

## Idea

- One bridge channel per pair of orchs that talk, such as `ly-bridge-neve-prime--osl-prime`, made by the machine that sends first; the other machine's bot joins it itself (`channels:join`) and the operator is invited, so the operator sees the exchange.
  This is the one exception to two apps never writing to one channel.
- Address format (operator, 2026-11-08): a name on the same machine stays as today, such as `ly:orch`; a name on another machine puts the machine and a double colon before it, such as `neve::ly:orch`. No name holds two colons in a row today, so it never reads as a `<hub>:<strand>` name, and it carries no shell glob character (brackets were dropped for that) and does not look like an e-mail address (`@` was dropped for that).
- Creel hands a post to a `<machine>::` name to the Slack relay instead of the machine record.
- The receiving relay delivers it as inbound mail from `neve::ly:orch`, never as the operator, and creel journals it as it journals mail from another hub.
- Proof of sender: the message came from the bot of a known machine's app (its app id); only the orch of a machine the operator approved is trusted, and the receiver weighs it as cross-hub mail, never as an operator instruction.
- Size: Slack and creel together, so a Motif with its own Discussion.
- Built after `slack-push-channels` lands; the operator wants it later, not now.
