---
round: 1
ledger:
  - key: adopt-gate-tiers-wiring-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: check-fixture-clock-constant
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: adopt-envelope-message-unasserted
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: bar-mark-before-clock-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four test-coverage findings (F1 to F4), all with local fixes the finding text spells out.
None is gating, so the loop converges.
