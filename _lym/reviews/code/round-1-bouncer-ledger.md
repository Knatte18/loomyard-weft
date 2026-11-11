---
round: 1
ledger:
  - key: embed-own-test-files-invisible
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: spawn-marked-selection-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: method-body-row-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d5-cases-without-row
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: embed-pattern-shapes-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: deleted-file-imports-miss
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dotless-import-treated-stdlib
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: doc-comments-lean-on-symbols
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
Four gating design findings at MEDIUM remain open: embed scope for the importing package's test files, a missing method-body row, D5 cases without a row, and unpinned go:embed pattern shapes.
The spawn-marked selection finding is a decision for the parent.
The remaining findings are LOW.
