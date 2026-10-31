---
round: 1
ledger:
  - key: hold-notices-task-owned-elsewhere
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: beam-status-help-omits-refused-start
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: held-task-max-runs-slot-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings, none gating.
F1 (`hold-notices-task-owned-elsewhere`) concerns the hold branch running before the ownership check.
F2 (`beam-status-help-omits-refused-start`) concerns stale `lym beam status` help text.
F3 (`held-task-max-runs-slot-untested`) concerns a missing `BuildView` row for the `max_runs` slot.
