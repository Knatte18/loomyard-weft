---
round: 1
ledger:
  - key: usage-limit-wake-polling-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: closed-workaround-sections-skip-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: status-report-cd-vs-no-cd-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: registry-exclusion-not-pinned
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, none gating.
F1 maps to `usage-limit-wake-polling-contradiction`, F2 to `closed-workaround-sections-skip-unrecorded` (deferred to the parent), F3 to `status-report-cd-vs-no-cd-rule`, F4 to `registry-exclusion-not-pinned`.
