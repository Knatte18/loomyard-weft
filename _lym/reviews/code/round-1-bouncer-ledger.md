---
round: 1
ledger:
  - key: part-label-duplicate-passes-check
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised F1: `vocabulary()` in `internal/raddleparser/check.go` does not refuse a part whose `labels` list one module label twice.
The board's `validateLabels` refuses that entry at Raddle-File, after approval.
The fix text names the check constant, its detail clause, the spec bullet and the `TestCheck` row, so it is mechanical.
