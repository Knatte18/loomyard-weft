---
round: 1
ledger:
  - key: restamp-failure-skips-bind-trace
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised one finding, F1: `postBatchChecks` returns the restamp error after a successful bind and before the bind trace is logged, a no-line case that card 5 and `doc.go` do not list.
It is LOW and non-gating.
