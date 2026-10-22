---
round: 1
ledger:
  - key: boundary-held-survives-turn-change
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: group-sigterm-failure-no-kill-fallback
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: group-terminate-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-trims-beyond-card-8
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, none recurring.
F1 (`boundaryHeld` not cleared when the writer leaves the boundary while covered) is the sole gating one.
F2 and F3 concern the Linux group-terminate path in `internal/proc`, and F4 the overview trims beyond card 8.
Each finding's fix is spelled out in the review, so all four are mechanical.
