---
round: 1
ledger:
  - key: failing-row-bound-from-below-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-overstates-exact-pin
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings.
F1 (`failing-row-bound-from-below-only`): the failing-start branch of `TestStartup_LoadTurnAPIError` lacks an upper bound.
F2 (`summary-overstates-exact-pin`): the change description claims an exact pin where the bound is a one-back-off window.
Neither is gating, so the loop converges.
