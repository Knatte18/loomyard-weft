---
round: 1
ledger:
  - key: no-status-liveness-halt-precedes-status-read
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-liveness-read-ends-call-instead-of-warning
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: halt-after-judge-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-liveness-read-inside-wait-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ended-at-entry-row-does-not-pin-keeps-session
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, with no earlier ledger.
F1 maps to `no-status-liveness-halt-precedes-status-read`, F2 to `failed-liveness-read-ends-call-instead-of-warning`, F3 to `halt-after-judge-untested`, F4 to `failed-liveness-read-inside-wait-untested`, F5 to `ended-at-entry-row-does-not-pin-keeps-session`.
Each fix text names its exact edit, so all five are mechanical.
