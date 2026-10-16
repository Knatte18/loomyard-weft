---
round: 1
ledger:
  - key: redundant-webster-wire-verify-tiers-row
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: unreachable-absent-verify-tiers-branch-in-load
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: listed-wire-row-never-carries-tiers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: no-route-for-two-tier-render-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-comment-on-loom-wire-load-call
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings against the plan, F1 to F5, in the order of the keys above.
F1 (BLOCKING) and F2 (design, MEDIUM) gate.
Each fix is a local text correction the finding already specifies.
