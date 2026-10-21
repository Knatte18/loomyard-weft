---
round: 1
ledger:
  - key: lyx-main-package-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: seenfiles-case-folded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 (`lyxMainPackage` has no test against go.mod's module path) and F2 (`seenFiles` keyed through the case-folding `normDir`).
Both are LOW and local, with fix text that fully determines the change.
No earlier ledger exists.
