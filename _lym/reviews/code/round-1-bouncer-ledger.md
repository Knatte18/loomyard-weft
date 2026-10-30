---
round: 1
ledger:
  - key: notice-memory-restart-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: run-selection-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: respawn-cap-stale-and-memory-error-hides-run
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: respawn-cap-declared-twice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: zero-quiet-min-disables-notice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: dead-reset-helpers-battenshed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: zero-quiet-window-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: readnoticechild-lock-fallback-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: delivery-attempt-cleanup-untested
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings, none recurring.
The gating pair is the missing tests for the notice memory across a dobby re-exec and for which runs the notices visit.
The rest are LOW or NIT: a stale respawn cap and memory-read error handling, a doubly declared respawn cap, the zero `notice_quiet_min` behaviour change, dead helpers, and three smaller test gaps.
