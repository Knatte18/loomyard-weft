---
round: 1
ledger:
  - key: drift-open-maps-lack-shape
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: whole-entry-and-flow-list-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: shapeof-tag-rules-partly-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: all-or-nothing-multi-pair-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
F1 is drift's capture and compare open maps lacking a declared shape.
F2 is the untested whole-entry and bracketed-value branches of `setShapedValue`; it is the one gating finding.
F3 is the partly exercised `ShapeOf` tag rules.
F4 is the untested all-or-nothing check on a multi-pair set.
Each fix is spelled out in the finding text, so each is mechanical.
