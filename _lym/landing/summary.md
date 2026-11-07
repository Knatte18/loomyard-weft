# slackengine: stop reposting a message Slack accepted

The relay posted an orch's reply to `#loomyard` every five seconds until `lym slack stop`.
`chat.postMessage` answers with `channel` as an id string, but the shared `response` type read it as the object `conversations.info` returns.
The decode failed after Slack had accepted the post, the read counted as failed, and creel redelivered the mail.

Closes the board task `slack-post-decode-repost`.

## What changes

- `call` in `internal/slackengine/api.go` returns the answer body once Slack accepted the call, reading only `ok` and `error` from it.
  `ConversationInfo` and `History`/`Latest` decode only the fields they use, each into its own answer type.
  `PostMessage` reads nothing past `ok`, so a post Slack accepted cannot fail on the shape of the rest.
- `internal/slackengine/outbound.go` no longer turns a failed outbox write into a read error that would repost accepted mail.
  A failed write keeps the record in memory in the file's place, and the read returns the write's error once it ends, so the relay log names it.
- `internal/slackengine/doc.go` and `docs/overview.md` state the new guarantee.

## Design decisions

- The in-memory record is lost when the relay process ends.
  A restart before a good write can therefore post that mail again, so the guarantee is "not posted again within one relay process", and the docs say so.
- `errOutboxWrite` and the rule that held the stale-mail line back after a failed write are removed.
  The in-memory record keeps the stale count at 0 once the line posts, and `TestOutboundRead` pins that the line posts once across two reads under a failing write.
- Tests decode a body from Slack's documented examples for each method lym calls: `auth.test`, `conversations.info`, `conversations.history`, `users.info` and `chat.postMessage`.

## The unclaimed reply from the live test

Reply #1023 from `ly:orch` (posted 2026-11-07T17:38:40Z) is still queued: `lym creel log --to ly:SLACK` shows it posted and not acknowledged.
The hub's `.lym/slack/` directory holds no `outbox.json`, so the relay has no record of it.
With this change, the next `lym slack start` posts it to the channel once and acknowledges it.
The operator has already seen it, so that is one duplicate at most.
Mail older than a day is acknowledged unposted, so leaving the relay stopped until after 2026-11-08T17:38:40Z also clears it without a post.

## To check by hand

- Start the relay against the real workspace and confirm one reply posts once and the relay log shows no decode error.
- Stale-mail-line coverage under a failing outbox write (`internal/slackengine/outbound_test.go`, `TestOutboundRead`): the review left this finding open, so check the new row fails if the line reposts.
