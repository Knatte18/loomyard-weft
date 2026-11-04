---
round: 1
ledger:
  - key: reap-children-unreaped-starts
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: startup-reap-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: list-deletes-zombie-leader-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: stale-never-waited-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-claims-not-borne-out
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: zombie-reading-pinned-only-by-fakes
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: isrunning-field-bound-to-isalive
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, none recurring.
F1 (`reap-children-unreaped-starts`) is the gating BLOCKING one, F2 (`startup-reap-untested`) the gating MEDIUM.
The rest are LOW or NIT.
