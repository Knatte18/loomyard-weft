# Slack in reed: a connection toggle and a presence toggle, presence per machine and switching by itself

`slack-presence-auto` · Note · feature, reed, shuttle, slack

## Seen

`slack-relay` landed (`5e9d4ffa`) without the operator's presence decision of 2026-11-05: presence changes only through `lym slack away|back`, the Slack texts and `lym slack stop`.
The Slack rebuild `slack-push-channels` (`92280f7e`) gives each orch its own channel and one Socket Mode relay per machine; presence is still per hub, and reed shows neither presence nor whether the relay is connected.

## Decision (operator, 2026-11-05 and 2026-11-08)

Two toggles on reed's status bar, beside `follow`, each lit when on, each redrawn when its state changes by any route:

- A connection toggle, lit while the machine's Slack relay is connected. A click starts or stops the relay, so Slack can be turned off without touching presence.
- A presence toggle, lit while presence is away (`slack`): the orchs forward everything for the operator to Slack and reach the operator there, not in the console. A click switches presence either way.

Presence is per machine, not per hub: away on a machine holds for every hub and orch on it (operator, 2026-11-08).

Presence also switches by itself:

- A message from the configured operator on Slack sets presence to away when the relay is connected and presence is `terminal`; lym sets it (the relay, before it forwards the message), not the orch's judgment, and the `[slack presence]` notice goes out as for any change.
- A line the operator types in an orch's pane while away sets presence back to `terminal`, set by lym from the `UserPromptSubmit` event shuttle already records.
- lym tells the operator's own lines from what lym puts in the pane itself: a slash command such as ORCHWATCH's `/compact` and `/color`, a text creel or another lym program injected (creel journals each inject), a task notification Claude Code submits itself, and mail a hook hands the session never switch presence.
  The design names how each kind is recognised, such as matching a submitted prompt against the inject creel journaled for that pane just before it, and tests each kind.

## Open

- The two toggles' labels (operator weighing `slack` and `relay`).
- How an orch keeps to Slack while away, so a compacted or forgetful orch does not answer only in the console. The operator's idea: a hook that catches an orch ending its turn with a console answer while away.
