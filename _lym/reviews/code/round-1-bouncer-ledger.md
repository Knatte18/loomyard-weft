---
round: 1
ledger:
  - key: failed-overview-render-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: up-to-date-overview-row-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two test-coverage findings against the hub config walk's HEDDLE.md render.
F1 (`failed-overview-render-untested`): no row runs the walk over a `heddle.json` that fails to load.
F2 (`up-to-date-overview-row-missing`): no row starts from an overview that already matches.
Both fixes are local test additions whose content the finding text already specifies.
