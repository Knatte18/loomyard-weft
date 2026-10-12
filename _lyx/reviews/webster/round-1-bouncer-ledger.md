---
round: 1
ledger:
  - key: kept-value-scanner-untested-on-escapes
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: stale-shared-lock-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: storedentry-missing-own-unmarshaljson
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 was the first round, with no prior ledger.
F1 maps to `kept-value-scanner-untested-on-escapes`, F2 to `stale-shared-lock-comments` and F3 to `storedentry-missing-own-unmarshaljson`.
None is gating, so the loop converges.
