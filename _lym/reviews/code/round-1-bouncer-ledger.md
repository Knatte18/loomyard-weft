---
round: 1
ledger:
  - key: pattern-lym-files-stale-hub-wide-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-stale-hub-wide-machine-override
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: openlaunch-danger-full-access-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings: two stale "hub-wide module" wordings (F1 in `pattern/PATTERN-lym-files.md`, F2 in `docs/overview.md`) and one missing `OpenLaunch` test row for `danger-full-access` (F3).
None is gating, and each has a fix its text fixes.
