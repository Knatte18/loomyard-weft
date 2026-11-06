---
round: 1
ledger:
  - key: plan-order-routed-through-fix-plan
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stencil-quotes-old-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: planglyph-test-readers-check-literal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all open.
F1 (`plan-order-routed-through-fix-plan`) is the gating one: the master stencil sends a `plan_order` refusal through `fix-plan`, which only reaches the plan writer.
F2 (`stencil-quotes-old-way-forward`) is the stencil quoting a way-forward wording the delete-before-reference case no longer has.
F3 (`planglyph-test-readers-check-literal`) is two test readers keeping the check-name literal.
Each fix text already fixes the result, so all three are mechanical.
