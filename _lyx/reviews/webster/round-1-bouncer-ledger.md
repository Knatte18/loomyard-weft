---
round: 1
ledger:
  - key: final-precedence-over-forked-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: no test sets `Forked` and `Final` on one gate entry, so reordering their assignments in `Run.evaluateGate` would pass the suite.
The fix is a local test case the finding text fully specifies.
