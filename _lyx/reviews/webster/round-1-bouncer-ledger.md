---
round: 1
ledger:
  - key: weft-absent-guard-untested-on-adopt-restore
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: subpath-anchor-scratch-link-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: wirejunctionswith-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: empty-exclude-path-in-report
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, none BLOCKING and none gating at MEDIUM or worse.
F1 and F2 ask for missing test rows, F3 for a doc comment update, F4 for a non-empty location in an exclude-failure report.
All four fixes are local corrections whose text the findings already fix.
