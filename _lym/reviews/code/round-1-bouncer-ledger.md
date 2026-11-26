---
round: 1
ledger:
  - key: oom-watch-wait-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1: the review approved with one LOW finding, F1.
No assertion in `TestEnterSession_DrivesTheToldTmux` depends on the cleanup wait covering the OOM watch goroutine.
The finding text specifies the assertion to add, hence `mechanical`.
