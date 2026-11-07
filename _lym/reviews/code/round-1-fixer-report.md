# Fixer-report, code round 1

## Changes

- **F1** (fixed as proposed): `TestOutboundRead` gained `StaleMailLineIsPostedOnceWhenTheOutboxWriteFails`.
  The batch holds one fresh and two stale messages, the outbox path turns into a directory at the first post, and the first acknowledgement is refused.
  Across two reads it asserts exactly one count line and that both reads return the failed outbox write.
  The count line is posted from the first read and the second read loads the in-memory record, so no second line follows.
- **F2** (fixed as proposed, one row implemented through a third read):
  - `OutboxWriteFailureRidesAlongWithAFailedPostAndWithTheEndedBudget` has two rows.
    `FailedPost` asserts the read error carries both the post failure and `outbox file`.
    `PostingBudgetEnded` asserts the `errReadBudget` return carries `outbox file`.
  - `OutboxFileWritableAgainStandsInPlaceOfTheRecordHeldInMemory` removes the directory, reads again with the budget ending the read, and asserts the read returns nil and the file holds the record.
    A third read then asserts each message was posted exactly once, which fails with the `unsaved` reset at `outbound.go:266` removed.
    This third read came from the self-check.
- **F3** (fixed with its own change): `docs/overview.md` now ends "a message Slack accepted is not posted again within one relay process".
  The proposed wording lacked the restart bound that F4 adds to `doc.go` and the `outbound.go` header, so the overview would have stated a stronger promise than the package.
- **F4** (fixed as proposed): `doc.go` gained the sentence that the in-memory record is lost when the relay process ends, so a restart before a good write can post that mail again.
  The `outbound.go` header says "Once Slack accepted a message, the relay posts it no more within one relay process", on a line of its own.
  The gate then flagged the header's first draft, which broke one sentence across two lines at a comma; it is now two sentences, one per line.

Gate: `lym gate test ./internal/slackengine` passed, 126 of 126 targets.

## Self-check

- fix-code-correctness, MEDIUM: the recovery test passed with the `unsaved` reset removed. Fixed as the lens proposed, a third read.
- fix-consistency, LOW: the overview line lacked the restart bound. Fixed.
- fix-proposal, LOW: the same recovery-test gap as fix-code-correctness. Merged and fixed with it.
- fix-proposal, LOW: the same unbounded overview wording as fix-consistency. Merged and fixed with it.
- fix-affected-set: no findings.

No fork returned without a report.

## Disputed

None.

## Deferred

None.
