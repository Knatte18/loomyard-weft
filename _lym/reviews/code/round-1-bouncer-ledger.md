---
round: 1
ledger:
  - key: empty-under-skips-negative-rows-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: singular-skip-wording-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: skip-count-includes-duplicates
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, all below the gating threshold.
F1 maps to `empty-under-skips-negative-rows-untested`, F2 to `singular-skip-wording-uncovered` and F3 to `skip-count-includes-duplicates`.
Each fix is a local correction whose text the finding already fixes.
