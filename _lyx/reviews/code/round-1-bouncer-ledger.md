---
round: 1
ledger:
  - key: recovery-retry-zero-count-order-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised F1, a LOW test-coverage gap: no `TestRecoveryRetry` row pins the zero-count early return behind the kind and terminal checks.
It is not gating and the fix is a local test-row addition.
