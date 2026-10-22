---
round: 1
ledger:
  - key: entry-doc-comments-omit-findings-chain
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised one finding: the doc comments of `multiLLMEntry`, `describeEntry` and `burlerReviewEntry` spell out the producer construction without the new findings-file chain.
It is a NIT and non-gating.
