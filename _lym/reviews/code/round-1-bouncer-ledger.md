---
round: 1
ledger:
  - key: findings-path-built-twice
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised one finding, F1, mapped to `findings-path-built-twice`.
The test helper `blockingFindingsRun` and its subtest each build the `pr-findings.md` path from the same literals.
The fix is local and fully specified: return the path from the helper and drop the second join.
