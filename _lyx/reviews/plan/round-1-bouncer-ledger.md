---
round: 1
ledger:
  - key: steplookupcontract-doc-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: runlock-kept-key-row-duplicates-act
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: saveorder-anchor-not-after-known-fields
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 was the first round, with no prior ledger.
It raised three non-gating consistency findings (F1, F2, F3) and no design finding.
