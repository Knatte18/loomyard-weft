---
round: 1
ledger:
  - key: cluster-test-marker-overclaims-enforcement
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1: a cluster entry carries `(test)` when any member was test-enforced, so clusters with review-only members claim more enforcement than they have.
The fix is local and its text is fixed by the finding: document the cluster meaning in `internal/pattern/doc.go` and mark the review-only members in the background files.
There is no earlier ledger, so nothing else carries forward.
