---
round: 1
ledger:
  - key: tier1-under-step-unlicensed
    rounds: [1]
    status: open
  - key: anomaly-pending-retry-unlicensed
    rounds: [1]
    status: open
  - key: failed-halt-envelope-drops-friction
    rounds: [1]
    status: resolved
  - key: halt-filing-intent-unbounded
    rounds: [1]
    status: resolved
  - key: reflect-settle-then-spawn-double-timeout
    rounds: [1]
    status: resolved
  - key: reflect-record-failures-unspecified
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-untested
    rounds: [1]
    status: resolved
  - key: drive-stop-report-friction-unlicensed
    rounds: [1]
    status: resolved
  - key: step-hook-impactsummary-misnames-consumers
    rounds: [1]
    status: open
---

Round 1, generation 0: there is no `_lyx/loom/rework/`, so the decision record is the whole answer key.

Two scope findings are still open, and both are BLOCKING.
Card 7 runs Tier 1 anomaly filing under `step`, although the decision record lists Tier 1 filing as `run`-only under Scope Out.
Card 8 adds a pending-retry marker that no part of the record licenses.
Parts of the overview and of cards 2, 6, 10 and 11 depend on card 7.
The fixer deferred both as an operator decision.
They close in one of two ways: the cards and their dependent parts are removed and a Shared Decision records that the addition is not carried, or the decision record is amended through Discussion.

The fixer resolved the failed-halt error envelope, the missing bound in card 6's Intent, the doubled timeout in card 1, the unspecified record failures, the untested acceptance criteria and card 9's unlicensed stop-report sentence.

One LOW finding is new this round: card 2's ImpactSummary names "lifecycle" as a `shedverbs` consumer.
The real consumers are `battencli` and `shedcli`.
