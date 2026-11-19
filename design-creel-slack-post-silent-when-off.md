# A creel post to SLACK reports ok while Slack is disconnected

`creel-slack-post-silent-when-off` · Task · bug, creel, slack

## What was seen

2026-11-19: `lym creel post ly:SLACK "<text>"` answered `{"ok":true}` (message 2035) while the operator had disconnected Slack.
`lym slack status` showed `relay_live: false`, `connected: false` and an empty `forwarding` list, so the post reached no channel, and the sender was not told.
The orch believed the result was on Slack.

## Idea

A post to SLACK while no relay runs, or while the sending orch has its slack button off, should say so:
refuse with the state and a way forward (`lym slack start`, or press the button), or answer ok with a `delivered: false` field and the reason.
The sender then reports the result in its pane instead.
