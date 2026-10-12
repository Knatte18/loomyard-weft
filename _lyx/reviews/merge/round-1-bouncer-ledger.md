---
round: 1
ledger:
  - key: alignment-attempt-deadline-ignores-pause-sleep
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: pralign-wait-parent-lock-pause-seam-mid-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised two findings.
The alignment attempt deadline (F1) is wall-clock and ignores pause and sleep, which is gating.
PR-Align's wait reads the parent-lock pause seam, whose meaning the task changed (F2), which is non-gating.
