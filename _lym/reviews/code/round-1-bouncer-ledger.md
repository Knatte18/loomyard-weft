---
round: 1
ledger:
  - key: body-edit-warning-guards-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stale-upsert-task-siblings-doc
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: board-skill-omits-upsert-warning
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: record-file-upsert-no-warning-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, F1 to F4, in the order of the keys above.
F1 is the only gating one: the `isDone` and `recordStored` guards of `bodyEditRecordWarning` have no test row that fails when a guard is dropped.
The other three are LOW: a stale method name in the boardengine doc, a board skill that documents the warning for `promote` only, and a missing assertion that the `--record-file` upsert carries no warning.
Each fix text already names its edit, so all four are mechanical.
