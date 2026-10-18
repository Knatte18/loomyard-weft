---
round: 1
ledger:
  - key: apply-failure-leaves-files-unrestored
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: push-failure-reported-as-commit-failure
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: layer-refusal-before-board-dir-refusal
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: apply-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: config-help-preserved-keys-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: missing-base-error-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: warn-unknown-keys-wholeentries-and-cost
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dry-run-from-pair-unpinned
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; no prior ledger exists.
Eight findings are open.
The one gating finding is the failed-apply restore gap in `reconcileDefaultLayer`, which F4's test gap also touches.
The rest are LOW or NIT local corrections.
