---
round: 1
ledger:
  - key: run-record-failures-key-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: failures-prefix-sibling-row-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: failed-tier-without-failures-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised three test-coverage findings, all class `scope`.
F1 maps to `run-record-failures-key-unpinned`, F2 to `failures-prefix-sibling-row-missing`, F3 to `failed-tier-without-failures-unpinned`.
None is gating, so the loop converged.
