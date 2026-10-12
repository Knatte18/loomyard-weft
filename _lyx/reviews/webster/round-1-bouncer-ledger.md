---
round: 1
ledger:
  - key: run-help-stale-pause-clear
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: stale-pause-comments
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised two LOW findings: F1 maps to `run-help-stale-pause-clear` and F2 to `stale-pause-comments`.
Neither is gating.
