---
round: 1
ledger:
  - key: move-weft-rolls-back-landed-commit
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: move-in-progress-misnames-lock-holder
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-own-marker-falls-back-silently
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW design findings, F1 to F3, with no previous ledger to carry forward.
F1 maps to `move-weft-rolls-back-landed-commit`, F2 to `move-in-progress-misnames-lock-holder`, F3 to `unreadable-own-marker-falls-back-silently`.
Each fix is local and its result is fixed by the finding text.
