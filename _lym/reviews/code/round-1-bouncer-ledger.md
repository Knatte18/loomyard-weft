---
round: 1
ledger:
  - key: writerfangaps-subtest-misnested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tautological-board-scope-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW `consistency` findings in `internal/lenses/config_test.go`.
F1 is the misnested `writerFanGaps` subtest; F2 is the tautological board-scope half of the `procedureLenses` assertion.
Neither is gating, so the loop converged in round 1.
