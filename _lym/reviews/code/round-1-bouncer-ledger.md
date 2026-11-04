---
round: 1
ledger:
  - key: config-layers-degrading-list-omits-ideconfig
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: task-placeholder-and-opentask-files-untested
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: ide-yaml-load-failure-untested
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: prime-path-untested
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: filestoopen-skip-branches-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: array-form-editor-associations-replaced
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ide-yaml-loaded-over-caller-stack
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so there is no earlier ledger to carry forward.
Seven findings, none BLOCKING.
Four MEDIUM findings are consistency-class: a PATTERN list that omits `ideconfig`, and three test gaps in `ideengine`.
Three LOW findings remain: one test gap, and two design-class findings, array-form editor associations and the config stack `ide.yaml` loads over.
Every fix is a local correction whose text the finding already fixes.
