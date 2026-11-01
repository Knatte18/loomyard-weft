---
round: 1
ledger:
  - key: stale-never-died-claims
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stop-branch-order-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 (`stale-never-died-claims`), whose fix adds the stop-marked exception to three doc statements, and F2 (`stop-branch-order-unpinned`), whose fix adds a table row to an existing test.
Both fixes are local and fully specified by the finding text.
