---
round: 1
ledger:
  - key: alignment-attempt-json-bypasses-state
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
---

Round 1 raised one finding, F1.
The parent's `internal/alignment` rewrites `attempt.json` through `os.WriteFile`, outside `internal/state`, so a rewrite drops unknown keys.
The task's `PATTERN-tolerant-records` forbids that.
