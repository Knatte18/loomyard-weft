---
round: 1
ledger:
  - key: d11-audit-table-incomplete
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: recovery-branches-unreached-by-tests
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised two MEDIUM findings.
F1 (`d11-audit-table-incomplete`): the D11 audit names a fixed set of loops and misses Info-or-above lines inside them and in other loops.
Its fix adds new card 10 behavior, so it needs a fresh review.
F2 (`recovery-branches-unreached-by-tests`): five once-per-episode guards have no test reaching their Info recovery line.
Its fix extends existing test rows only.
