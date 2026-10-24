---
round: 1
ledger:
  - key: dead-end-refuses-amendment-at-cap
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: git-fault-read-as-dead-end
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dead-end-missing-negative-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: recordbatch-test-split-doc-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: waiting-batch-misreported-at-exit
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: stencil-two-sentences-one-line
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, none MEDIUM or worse.
The two `design` findings (F1, F2) concern the run-entry dead-end check and are LOW, so the loop converges.
