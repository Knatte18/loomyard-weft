---
round: 1
ledger:
  - key: reap-root-resolution-outcomes-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding, F1: `TestReap` pins only the resolved-link-target outcome of `rootProcesses`.
The written-root-under-a-linked-root and unresolved-root outcomes have no row.
The finding text names the exact rows to add, so the fix is mechanical.
