---
round: 2
focus:
  - "internal/slackengine/outbound.go, the outbound read handler (`post` and `reportStale`): when a read skips mail older than 24 hours and a later message of the same read fails to post or its outbox record write fails, is the one line naming how many messages were skipped still posted exactly once, with the right count, on the redelivered read?"
---

The focus follows the one gating finding of the previous round.
