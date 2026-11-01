---
round: 1
ledger:
  - key: orch-rules-file-dropped-silently
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: changestore-error-warning-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings, F1 and F2, both fixes local and fully specified by the finding text.
F1 asks for one sentence in the rules command help and the orch role text about the no-longer-read `_lym/orch/rules.md`.
F2 asks for tests pinning the store-error and warning returns of `ChangeStore` through `writeOrchRules`.
Neither is gating, so the loop converged in round 1.
