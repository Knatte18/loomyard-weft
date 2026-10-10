---
round: 1
ledger:
  - key: partitioned-member-consumers-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
---

Round 1 raised F1: the partitioned-member behavior added to four consumers of `answerSymbols` (`buildEditRegions`, `coverageSubjects`, `coveredByTarget`, the `resign-interface-method` arm) has no test.
It is non-gating, so the loop converged with it open.
