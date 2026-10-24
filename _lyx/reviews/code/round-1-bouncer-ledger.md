---
round: 1
ledger:
  - key: waiterless-run-key-leak-on-deleted-dir
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised only F1, a NIT: a run directory deleted mid-episode leaves its key in `waiterlessRuns`.
The reviewer judged it bounded and like `unreadableFiles`, so no change is needed.
