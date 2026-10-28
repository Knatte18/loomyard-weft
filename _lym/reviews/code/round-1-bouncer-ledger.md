---
round: 1
ledger:
  - key: start-failed-cause-not-decided-form
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: trace-fallback-requires-start-failure
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cancelled-backoff-reports-phantom-tries
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-path-fallback-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: installbinary-rename-failure-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
F1 (the start-failed cause lacks the decided `after <n> tries` form) and F2 (the loop-trace fallback wrongly requires a start failure) gate.
F3 (a cancelled backoff reports tries never made) is fixed together with F1.
F4 and F5 are missing test rows for the empty-path fallback and the `installBinary` rename failure.
