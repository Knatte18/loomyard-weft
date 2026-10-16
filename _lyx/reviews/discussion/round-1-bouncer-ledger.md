---
round: 1
ledger:
  - key: d6-sixty-second-bound-ignores-base-reset
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-confirm-interval-vs-changed-set-reset
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: d4-last-connect-fails-against-lock-holding-winner
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-gate-seam-bound-omits-bypass-gate-and-callers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-cut-test-differs-from-fold-skip-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-docs-list-misses-package-docs
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all still open.
Three are design-class at MEDIUM and gate: D6's 60 s bound under new mail or a failed listing, D4's last connect against a winner that holds the lock but does not yet listen, and D9's gate seam bound, which omits the bypass-permissions gate and the seam's callers.
The D6 confirm interval versus the changed-set base reset is a MEDIUM decision finding and closes with a rewording.
The D1 cut test and the Scope doc list are LOW and mechanical.
