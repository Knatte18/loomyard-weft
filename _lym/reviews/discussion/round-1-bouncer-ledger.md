---
round: 1
ledger:
  - key: d2-viewer-end-state-no-mechanism
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: parent-commit-7c23591-contradicts-design
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: d3-grouped-strand-placed-apart-ambiguous
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-criterion-condition-differs-from-decision
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-window-name-collision-facts-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-hard-coded-choice-cited-out-of-scope
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1, first ledger.
F1 (`d2-viewer-end-state-no-mechanism`) is the gating finding: D2's move leaves the viewer off the overview window and no step selects it.
F2 (`parent-commit-7c23591-contradicts-design`) is a parent question, not a fixer task.
F3 to F6 are local corrections to D3, D4 and the Constraints bullet.
