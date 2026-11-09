---
round: 1
ledger:
  - key: arras-run-test-misses-goto-rearm
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: arras-recipe-test-duplicates-coverage
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: split-config-table-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-batten-lists-omit-arras
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-split-keys-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five LOW findings: three test-economy and coverage gaps (F1-F3) and two documentation gaps in `docs/overview.md` (F4-F5).
None gates, so the loop converged on its first round.
