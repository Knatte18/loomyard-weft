---
round: 1
ledger:
  - key: compact-cycle-previous-baseline-recording-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: clear-reload-baseline-carryover-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: fresh-launch-pointermarksince-zeroing-unasserted
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW scope findings, all missing test coverage for `PointerMarkSince` handling: the requested compact cycle (F1), the /clear reload (F2) and `ResetForFreshLaunch` (F3).
The code-correctness lens found no defects.
