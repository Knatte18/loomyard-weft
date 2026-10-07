---
round: 1
ledger:
  - key: committed-tokencount-binary
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: reference-rule-test-missing-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: calibrate-peak-position-pricing-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: batcher-cost-before-shadowed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: reference-rule-test-redundant-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

First round, so no prior ledger.
Five findings are open: the committed `tools/tokencount/tokencount` binary (F1), the missing `t.Parallel` in `TestReferenceRule_Match` (F2), the untested per-position peak pricing in calibration (F3), the shadowed `before` in `internal/batcher/cost.go` (F4) and the redundant rule-test row (F5).
