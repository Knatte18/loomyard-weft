---
round: 1
ledger:
  - key: repeated-conflict-halt-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: stale-conflict-record-routes-other-stuck
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: rerouted-round-inherits-hard-cap
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: repeat-check-comparison-unspecified
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d1-rebegun-batch-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 of the discussion review raised five findings (F1 to F5), all open.
F1 maps to `repeated-conflict-halt-no-way-forward`, F2 to `stale-conflict-record-routes-other-stuck`, F3 to `rerouted-round-inherits-hard-cap`, F4 to `repeat-check-comparison-unspecified`, F5 to `d1-rebegun-batch-wording`.
F2 and F4 change a decision (record lifecycle, comparison key), so they need a fresh review.
