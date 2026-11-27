---
round: 1
ledger:
  - key: summary-remaining-gap-overstates-plan-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1: the review reports an empty review range, which it finds consistent with the task being settled as no code and no test.
Its one finding, F1, is that the `Remaining gap` section of `_lym/landing/summary.md` claims the plan rule covers more cards than it binds;
the finding gives the replacement text.
