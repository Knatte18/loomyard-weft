---
round: 1
ledger:
  - key: hookwake-zero-turnendedat-row-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: hookwake-test-doc-omits-hold
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings and nothing gating.
F1 asks for one test row pinning the zero `TurnEndedAt` hold.
F2 asks for the `TestScheduler_HookWake` doc comment to name the new hold.
Both fixes are local and fully specified by the finding text.
