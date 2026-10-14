---
round: 1
ledger:
  - key: d2-bound-claims-skips-none
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d2-guard-dir-drop-too-wide
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-bound-split-not-produced-by-covers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-package-level-ambiguous
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-scan-error-persists
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-fork-may-loosen-guard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-walk-differs-from-go-list
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-d3-rationale-rejected-alternatives
    rounds: [1]
    status: open
    class: decision
    severity: NIT
    fix: mechanical
  - key: support-log-q6-q7-order
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings on the discussion pair, none recurring since there is no earlier round.
Three are gating: the guard-directory drop rule is wider than its purpose, a persistent scan error can silently disable guard coverage for a run, and D4 does not bound a fork loosening a guard.
The rest are local text corrections and two NIT-level relocations.
