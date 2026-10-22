---
round: 1
ledger:
  - key: glyph-rejected-uncheckable-branch-untested
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: rename-disk-route-row-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: is-disk-checked-target-no-direct-test
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: same-name-declaration-list-unsorted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: disk-lookup-enotdir-errors
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: joined-table-rows
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings (F1 to F6, in the order above), none gating.
All are local corrections whose fix text names the exact change.
