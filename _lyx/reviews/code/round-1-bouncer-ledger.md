---
round: 1
ledger:
  - key: adoptcli-doc-stale-last
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: adopt-prime-only-suite-start-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: suite-log-path-inline-join
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings, all non-gating.
F1 maps to `adoptcli-doc-stale-last`, F2 to `adopt-prime-only-suite-start-untested`, F3 to `suite-log-path-inline-join`.
No earlier ledger exists.
