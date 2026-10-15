---
round: 1
ledger:
  - key: card2-unreachable-empty-files-guard
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card2-unreachable-no-scope-skip
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card1-refmember-rename-only-typing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings and no gating ones.
F1 (`card2-unreachable-empty-files-guard`) and F2 (`card2-unreachable-no-scope-skip`) both ask card 2 to drop guards and test rows for inputs that cannot reach `ScopeGuard`.
F3 (`card1-refmember-rename-only-typing`) asks card 1 to add an `Edit` target beside the `Rename` for the nil-plan default.
