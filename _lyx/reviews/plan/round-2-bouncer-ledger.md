---
round: 2
ledger:
  - key: test-budget-not-raised
    rounds: [1]
    status: resolved
  - key: template-comment-names-ttl-vocabulary
    rounds: [1]
    status: resolved
---

Round 1 raised two findings: a test-budget gap (`test-budget-not-raised`, BLOCKING design) and a TTL vocabulary leak into a `template.yaml` comment (`template-comment-names-ttl-vocabulary`, LOW consistency).
Round 2 found nothing; the reviewer checked the budget rows and the vocabulary containment and judged both clean, so both keys are resolved.
