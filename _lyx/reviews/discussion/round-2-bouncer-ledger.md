---
round: 2
ledger:
  - key: d6-sixty-second-bound-ignores-base-reset
    rounds: [1, 2]
    status: resolved
  - key: d6-confirm-interval-vs-changed-set-reset
    rounds: [1, 2]
    status: resolved
  - key: d4-last-connect-fails-against-lock-holding-winner
    rounds: [1, 2]
    status: resolved
  - key: d9-gate-seam-bound-omits-bypass-gate-and-callers
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d1-cut-test-differs-from-fold-skip-test
    rounds: [1, 2]
    status: resolved
  - key: scope-docs-list-misses-package-docs
    rounds: [1, 2]
    status: resolved
  - key: d6-wait-after-confirm-sweep-open
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: d6-failed-listing-pending-no-criterion
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-gate-seam-caller-bound-unanchored
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: d4-bound-overstates-nonrace-exit
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's four MEDIUM findings and two LOW findings are settled in round 2, except the D9 gate seam, whose bypass-gate premise remains as a LOW design finding.
Round 2 adds four findings: the wait after a confirm sweep, a missing acceptance criterion for D6's failed-listing rule, what holds D9's single-caller bound, and a wording nit on D4's Bound.
All are local corrections whose text the review already fixes.
