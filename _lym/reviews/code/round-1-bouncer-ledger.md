---
round: 1
ledger:
  - key: project-file-replay-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: replay-change-outside-record-scope
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: unknown-dependency-after-cell-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: project-draft-task-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: project-prefix-declared-twice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, F1 to F5, in the order of the ledger.
F1 gates: the project-file handling in the records replay has no test.
F2 is deferred to the run's parent, which records or refuses the replay change with `lym loom decision add`.
F3 to F5 are LOW gaps in goldens and a duplicated prefix constant.
