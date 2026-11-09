---
round: 1
ledger:
  - key: keywayforward-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: keywayforward-doc-garbled
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two findings, both consistency class.
F1 (`keywayforward-untested`) asks for a table test pinning the add, drop and sort-only messages of `keyListWayForward`.
F2 (`keywayforward-doc-garbled`) asks for a reworded doc comment.
Neither gates.
