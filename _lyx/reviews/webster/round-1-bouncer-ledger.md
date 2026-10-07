---
round: 1
ledger:
  - key: comment-line-breaks-semantic
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
---

Round 1: F1 found changed Go comments hard-wrapped mid-sentence, and one with two sentences on one line, in `internal/fabricengine` (`checkout.go`, `mutation.go`, `reconcile.go`) and `internal/landingshed/finalize.go`.
Every other focus question the reviewer examined was settled without a finding.
