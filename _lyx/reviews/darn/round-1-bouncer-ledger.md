---
round: 1
ledger:
  - key: pattern-test-speed-gate-strip-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: redundant-refresh-stored-existing-entry-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW consistency findings, both with a local fix the finding text already fixes.
F1 maps to `pattern-test-speed-gate-strip-overstated`, F2 to `redundant-refresh-stored-existing-entry-test`.
