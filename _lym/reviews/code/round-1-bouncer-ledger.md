---
round: 1
ledger:
  - key: battencli-activity-test-stale-liveness-unproven
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shuttle-anchor-refusal-blames-hidden-for-all-values
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW consistency findings, F1 and F2, both with local fixes their text fully specifies.
No finding is gating, so the loop converges.
