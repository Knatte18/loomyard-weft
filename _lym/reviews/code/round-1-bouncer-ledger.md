---
round: 1
ledger:
  - key: owed-push-way-forward-removed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: clone-refusal-test-lost-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: bolt-checkout-remote-board-adoption-lost
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: two-repo-cherry-pick-seam-test-deleted
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: overview-thresholds-warn-check-widened
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: mergestateactive-rationale-dropped
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: merge-danger-blast-radius-too-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so no earlier ledger exists.
The removal is a faithful revert of the one-repository work, but it also undid general two-repository behaviour and coverage.
F1 (owed-push way forward, BLOCKING) and F3 (Bolt checkout adoption, MEDIUM) are design findings whose fixes restore removed behaviour, so they need a fresh review.
F2, F4 to F7 are local restorations whose text the findings fix.
