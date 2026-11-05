---
round: 1
ledger:
  - key: reportconfig-overview-lines-uncarried
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
F1 is the only finding: card 08's Intent should also name the `docs/overview.md` package-tree line and the report module line's package list for `internal/reportconfig`.
The fix is a local Intent extension with no target change.
