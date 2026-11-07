---
round: 1
focus:
  - "internal/slackengine/outbound.go `save`, `loadOutbox`, `read`: does the in-memory `unsaved` record stay consistent across reads? Check that it clears or goes stale correctly after a later successful write, after `save(outboxRecord{})` on a confirmed acknowledgement, and when a failed write leaves an empty or older record standing in for the file."
  - "internal/slackengine/outbound.go `read` switch: does `saveFault` reach the caller on every path, including the `errReadBudget` and rate-limit paths, and is it ever returned alongside a successful acknowledgement in a way that makes the relay loop retry or back off wrongly?"
  - "internal/slackengine/outbound.go `post`, `postMessages`, `refused`: with write errors no longer returned, can any path still repost a message Slack accepted (a failed post of the stale line, a drop after `refusalCap`, a redelivered batch)? Settle whether the removed `errOutboxWrite` handling leaves a gap."
  - "internal/slackengine/api.go `call` and the per-method decoders: does every method lym calls decode in the shape Slack sends, and can a body Slack accepted still fail on a field the client reads (`History`, `Latest`, `ConversationInfo`, `UserInfo`)? Check the `ok: true` path for each method."
  - "Tests in api_test.go and outbound_test.go: does each of the five methods have a test on Slack's documented answer body, and does the outbox test fail without the fix? Check against the test-economy rules for duplicated or per-helper coverage."
  - "Landing summary `_lym/landing/summary.md`: check its claims against the diff, especially the in-memory record's lifetime, the removed `errOutboxWrite` rule, and the instructions for clearing reply #1023 (outbox.json content and the day-old rule against `staleMailAge`)."
  - "Doc and comment compliance in doc.go, api.go and outbound.go: the new prose follows the repository's own comment conventions and the package doc matches the code's behavior."
---

Seed focus for round 1, set from the decision record's three wants: per-method decoding, no repost after acceptance, and the landing note on the queued reply.
