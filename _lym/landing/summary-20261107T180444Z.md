# Slack relay: decode answers per method, never repost an accepted post

The first live Slack test showed the relay posting one orch reply to the channel every five seconds.
Slack accepted each post, but `chat.postMessage` answers with `channel` as an id string,
and the client decoded every answer into one shared type whose `channel` was the `conversations.info` object.
The decode failed after the post went through, the outbound read released the mail, and the next read posted it again.

## What changes

- **One shape per method** (`internal/slackengine/api.go`).
  `call` now reads only `ok` and `error` of every answer and hands back the body once Slack accepted the call.
  `ConversationInfo` and the history reads decode their own answer types;
  `AuthTest`, `UserInfo` and `PostMessage` read nothing past `ok`.
  A post Slack accepted can no longer fail on the shape of its answer.
- **An accepted post is never retried** (`internal/slackengine/outbound.go`).
  A post counts as handled once Slack accepts it, whatever fails after it.
  The remaining failure after acceptance was a failed write of the outbox record, which released the read and reposted the message.
  A failed write now keeps the record in memory in the file's place:
  the read goes on, the mail is acknowledged, and the read returns the write's error so the relay log names it.
  A redelivered batch reads the in-memory record and skips what was posted.
  A retry happens only when Slack did not accept the post.
- The package doc states both rules.

## Design decisions

- The client reads only the fields lym uses rather than modelling each answer whole.
  Slack adds fields freely, so a full model would invite the same class of decode failure.
- The in-memory record lives as long as the relay process.
  A relay that dies after a failed write and before the next good one can still repost on restart.
  That bound already held for a crash between the post and the write, and closing it would need a store other than the file.
- `errOutboxWrite` and the rule that the stale-mail line waits after a failed write are gone:
  with the record kept in memory, the line cannot post twice within the process.

## Tests

- `TestAPI/EachMethodSendsItsRequestAndDecodesSlacksDocumentedAnswer` answers each method lym calls
  (`auth.test`, `conversations.info`, `conversations.history`, `users.info`, `chat.postMessage`) with the example answer in Slack's API reference.
  It checks the decoded channel and messages and the follow-up page request the documented `next_cursor` causes.
  Before the fix, `PostMessage` failed on the documented body.
- `TestOutboundRead/OutboxWriteFailingAfterAnAcceptedPostAcknowledgesTheMailAndNeverRepostsIt` makes every outbox write fail after the post.
  With the acknowledgement confirmed, and with it refused and the batch redelivered, the message posts once and is acknowledged once.

## The reply left from the live test

The orch's reply is creel message #1023 to `ly:SLACK`, posted 2026-11-07T17:38:40Z.
`lym creel log --to ly:SLACK` shows it posted and never acknowledged, and the hub has no outbox file, so it is still queued.
The next `lym slack start` on a binary with this change posts it once more and acknowledges it;
the decode no longer fails, so it is not posted again after that.
The operator has already seen it three times or more, so that one post is a repeat.
To skip it, do one of the following before the next start:

- Write `_bolt/.lym/slack/outbox.json` as `{"handled":[1023]}`.
  The relay then acknowledges #1023 without posting it.
- Start the relay after 2026-11-08T17:38:40Z.
  The relay then acknowledges the reply unposted as mail over a day old and posts one line counting it.
