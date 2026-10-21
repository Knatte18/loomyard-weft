---
round: 1
ledger:
  - key: strip-commit-marker-moved-upstream-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding (F1): no test replays the task's strip commit, which adds the base-sparse marker, over a moved upstream through the parent's pull-first replay.
It is LOW and `scope`, so it does not gate.
