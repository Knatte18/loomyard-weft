---
round: 1
ledger:
  - key: pull-upstream-anchor-stop-no-anchored-row
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding, F1: the pull path's upstream-anchor stop has no test row in a hub anchored at a subdirectory.
The finding text names the table, the fixture and the assertion, so the fix is a local test addition.
