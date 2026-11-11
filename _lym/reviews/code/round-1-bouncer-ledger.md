---
round: 1
ledger:
  - key: await-forks-dispatch-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: resolvewithin-branches-without-row
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 (`await-forks` dispatch and weft-cwd run untested, MEDIUM) and F2 (`ResolveWithin` branches without a row, LOW).
Both are `scope` class, so neither gates; the loop converged in its first round.
