---
round: 1
ledger:
  - key: repeated-name-literals-named-constants
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: failed-compose-errors-round
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: zero-assist-timeout-escapes-cap
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: wait-error-ending-no-test-row
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: await-cap-resolution-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: per-helper-test-duplicates-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-omits-assist-config-keys
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, all open.
The gating ones are the repeated name literals (BLOCKING), the assistant seat built with blank instruction paths after a failed composition, and a zero assist timeout that bypasses the cap.
Each fix is a local correction its finding text already fixes.
