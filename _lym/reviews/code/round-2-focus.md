---
round: 2
focus:
  - "internal/slackengine/outbound_test.go, TestOutboundRead outbox-write-failure rows: with mail older than `staleMailAge` in the batch and the outbox path made unwritable after an accepted post, does the stale-mail count line post exactly once across two reads?"
  - "internal/slackengine/outbound.go read and save paths: are the joined post-and-save fault, the `errReadBudget` return and the clearing of `unsaved` after a later good write each exercised by a test?"
---

Both entries name where the diff removed a rule (`errOutboxWrite` and the stale-line hold-back) and ask whether the replacement behavior is covered.
