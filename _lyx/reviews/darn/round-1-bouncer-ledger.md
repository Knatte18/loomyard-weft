---
round: 1
ledger:
  - key: filled-keys-log-filter-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pernamemaps-field-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reconcile-help-lacks-dropped
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
F1 maps to `filled-keys-log-filter-untested`, F2 to `pernamemaps-field-comment-stale`, F3 to `reconcile-help-lacks-dropped`.
Only F1 gates.
