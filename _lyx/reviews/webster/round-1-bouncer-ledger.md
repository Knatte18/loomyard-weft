---
round: 1
ledger:
  - key: dry-run-takes-lock-and-creates-dir
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised one LOW consistency finding (F1): the unwire dry run, and the board prune and retire-legacy dry runs, take a lock and may create a directory before the no-write return.
This is the first round, so there is no earlier ledger to carry forward.
