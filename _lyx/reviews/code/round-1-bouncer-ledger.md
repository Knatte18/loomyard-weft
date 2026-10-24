---
round: 1
ledger:
  - key: relaunch-entered-row-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: midrun-tree-change-record-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: filter-exemption-rows-incomplete
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-yaml-doc-examples-old-tags
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: diff-base-by-written-time
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pruning-counts-args-entries
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings (F1 to F6, in the order above).
F1 is the only gating one: design class at MEDIUM.
All fixes are local corrections the finding text spells out.
