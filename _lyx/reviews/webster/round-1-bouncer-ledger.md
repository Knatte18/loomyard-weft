---
round: 1
ledger:
  - key: tokencount-left-out-line-singular
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised one finding, F1: the left-out line in `tools/tokencount/count.go` renders "1 session transcripts were left out" for a count of one, and `time_integration_test.go` pins that wording.
It is a non-gating NIT.
There was no previous ledger to carry forward.
