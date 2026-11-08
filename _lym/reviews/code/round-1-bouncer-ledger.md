---
round: 1
ledger:
  - key: presence-doc-comments-say-hub
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hook-failures-leave-no-trace
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reedengine-testmain-guard-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings (F1, F2, F3), none gating.
F1 maps to `presence-doc-comments-say-hub`, F2 to `hook-failures-leave-no-trace`, F3 to `reedengine-testmain-guard-untested`.
