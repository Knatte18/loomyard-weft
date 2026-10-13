---
round: 1
ledger:
  - key: sandbox-suite-names-view-button
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: batten-doc-stale-content-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: set-content-rows-no-caller
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: skip-when-box-equals-dead
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four LOW findings, all leftovers of the window-order change: a stale suite step (F1), a stale package-doc sentence (F2), a caller-less `SetContentRows` (F3) and a dead `SkipWhenBoxEquals` option (F4).
Each fix is local and fully stated by its finding.
No earlier ledger exists.
