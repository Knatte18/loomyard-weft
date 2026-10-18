---
round: 1
ledger:
  - key: reserved-by-read-path-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: post-help-stale-for-carried-mail
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-creel-entry-omits-carried-mail
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: released-read-retypes-notice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hold-owner-and-inject-order-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
The gating one is the missing test for the reserved-by-a-read path (F1).
The rest are a stale post help (F2), a stale overview entry (F3), a bounded retype after a released read (F4) and unpinned hold-owner and inject-order behavior (F5).
Each fix is a local correction its finding text already fixes.
