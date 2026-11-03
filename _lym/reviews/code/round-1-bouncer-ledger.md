---
round: 1
ledger:
  - key: seatengine-entry-points-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: burlerengine-engine-choice-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: burler-explore-engine-block-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three test-coverage findings, all class `scope`.
F1 maps to `seatengine-entry-points-unpinned`, F2 to `burlerengine-engine-choice-unpinned`, F3 to `burler-explore-engine-block-uncovered`.
None is gating, so the loop converged in the first round.
