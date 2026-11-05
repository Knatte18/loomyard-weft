# Slack presence follows where the operator writes, with a reed toggle

`slack-presence-auto` · Note · feature, reed, shuttle

## Seen

`slack-relay` landed (`5e9d4ffa`) without the operator's presence decision of 2026-11-05.
Its change description says under "Not included" that presence switching by itself and the reed toggle are not in the diff: presence changes only through `lym slack away|back`, the Slack texts and `lym slack stop`.

## Decision (operator, 2026-11-05)

Presence switches by itself, as well as by `away`, `back` and `lym slack away|back`.

- A message from the configured operator on Slack sets presence to `slack` when it is `terminal`.
  lym sets it (the relay, before it forwards the message), not the orch's judgment; the message is then forwarded as usual, and the `[slack presence]` notice goes out as for any change.
- A line the operator types in an orch's pane while presence is `slack` sets it back to `terminal`, set by lym from the `UserPromptSubmit` event shuttle already records.
- lym tells the operator's own lines from what lym puts in the pane itself.
  A slash command such as ORCHWATCH's `/compact` and `/color`, a text creel or another lym program injected (creel journals each inject), a task notification Claude Code submits itself, and mail a hook hands the session never switch presence.
  The design names how each kind is recognised, such as matching a submitted prompt against the inject creel journaled for that pane just before it, and tests each kind.
- Presence shows plainly in reed: a toggle on the status bar beside `follow`, labelled such as `slack: on` lit and `slack: off` unlit, which the operator can click to switch presence either way.
  It redraws when presence changes by any route.
- While presence is `slack`, the orch mirrors what it means for the operator to Slack; while `terminal`, it answers on Slack only mail that came from Slack.
