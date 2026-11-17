---
round: 1
ledger:
  - key: master-recover-batch-keeps-secret
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: added-files-net-tree-delta
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: new-tests-skip-t-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: correctness-finding-hides-secret-check
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: hub-patterns-untested-through-wiring
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reset-doc-claims-file-left-on-disk
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refused-commit-index-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: newpathsin-empty-pathspec-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: report-status-independence-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shipped-first-row-cannot-fail
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review; no earlier ledger exists.
Keys map to F1 through F10 in order.
The two design-wrong keys change a stencil rung and a gitrepo method contract, so their fixes need a fresh review.
