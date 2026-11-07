# Slack relay reposts a sent message: chat.postMessage answer fails to decode

`slack-post-decode-repost` · Task · bug, slack · running · Drift-Verify

## Seen (operator and orch, 2026-11-07, first live Slack test)

The orch replied to the operator's first channel message, and the relay posted the reply to `#loomyard` again every five seconds until the orch ran `lym slack stop`; the operator saw it three times or more.
`_bolt/.lym/slack/relay.log` shows each attempt failing after the post went through:
`outbound: slack: decode the chat.postMessage answer: json: cannot unmarshal string into Go struct field response.channel of type slackengine.ChannelInfo`.
`response` in `internal/slackengine/api.go` types `channel` as the object `conversations.info` returns, but `chat.postMessage` answers with the channel id as a string.
The tests ran against a fake Slack that never sent the real answer shape.

The relay is stopped; the reply is likely still unclaimed and would post again at the next `lym slack start`.

## Wanted

- Each Slack method's answer decodes into a shape that matches what Slack sends, with a test on a real answer body for each method lym calls (`auth.test`, `conversations.info`, `conversations.history`, `users.info`, `chat.postMessage`), taken from Slack's documented examples.
- A post Slack accepted (`ok: true`) is never posted again, whatever fails after it: an error after acceptance marks the mail delivered and logs the fault, and a retry happens only when the post was not accepted.
- Say in the landing whether the unclaimed reply from this test is still queued, and how it is cleared or delivered once, so the next `lym slack start` posts nothing twice.
