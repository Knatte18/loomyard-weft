# Let orchs on two machines talk through a Slack bridge channel

`slack-machine-bridge` · Task · feature, slack, creel, orch

## Seen (operator, 2026-11-08)

The operator wants the orchs on two machines (neve and OSL-1046) to be able to talk to each other through Slack, both machines connected at once.
The Slack relay rebuild (`slack-push-channels`, built from the Sketch `slack-v2`) keeps one app per machine, never lets two apps write to one channel, and treats every inbound Slack message as the operator's.
Creel today carries mail between the hubs of one machine only: a name `<hub>:<strand>` resolves through the machine record, and the receiving daemon confirms the sender by asking the sending daemon, which cannot work across machines.

The operator judged on 2026-11-11 that this is more useful than the relay's phone use alone, and that it should need no setup beyond Slack itself.

## Idea

- One bridge channel per pair of orchs that talk, such as `ly-bridge-neve-prime--osl-prime`, made by the machine that sends first; the other machine's bot joins it itself (`channels:join`) and the operator is invited, so the operator sees the exchange.
  This is the one exception to two apps never writing to one channel.
- Address format (operator, 2026-11-08): a name on the same machine stays as today, such as `ly:orch`; a name on another machine puts the machine and a double colon before it, such as `neve::ly:orch`. No name holds two colons in a row today, so it never reads as a `<hub>:<strand>` name, and it carries no shell glob character (brackets were dropped for that) and does not look like an e-mail address (`@` was dropped for that).
- Creel hands a post to a `<machine>::` name to the Slack relay instead of the machine record.
- The receiving relay delivers it as inbound mail from `neve::ly:orch`, never as the operator, and creel journals it as it journals mail from another hub.
- Proof of sender: the message came from the bot of a known machine's app (its app id); only the orch of a machine the operator approved is trusted, and the receiver weighs it as cross-hub mail, never as an operator instruction.
- Size: Slack and creel together, so a Motif with its own Discussion.
- `slack-push-channels` has landed (`92280f7e`: Socket Mode push, one channel per orch, two tokens), so the groundwork is there; the operator still wants this later, not now.

## Changed by `slack-per-orch-activation` (board tidy, 2026-11-11, landed as `0091f39c`)

- Each orch now has its own `slack` button on reed's status bar (the prime's orch and each beam's orch; a pair's orch has none), and has no channel until it is first pressed.
  The relay runs only while a button is on; the machine-wide connection switch is gone.
  The state is `forwarding.json` under the hub's `.lym/slack/`, and the channel map is `orch-channels.json`.
- A new orch channel is named `<machine>-<shortname>-<project handle>-<project slug>`, such as `neve-ly-p0-root`.
  A bridge channel's name above is a separate scheme and still open.
- Mail posted to `SLACK` by an orch whose button is off is dropped, and the `[slack presence]` notice is gone.
  A bridge design should follow the buttons: whether a bridge channel needs its own switch is open.
- Everything inbound is still treated as the operator's, and still one app per machine.
