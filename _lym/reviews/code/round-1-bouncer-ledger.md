---
round: 1
ledger:
  - key: fslink-scan-dot-import-of-os-skipped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: spawn-scan-allowlist-consulted-before-findings
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised two gating findings, F1 and F2, both `design` at MEDIUM.
F1 maps to `fslink-scan-dot-import-of-os-skipped`: `linkCallFailures` skips a file whose `os` import is a dot import.
F2 maps to `spawn-scan-allowlist-consulted-before-findings`: `headlessSpawnFailures` checks the allowlist before any finding exists, so a stale entry never reports, and no fixture row covers admission.
The review's fix text fixes the result of each, so both are `mechanical`.
No ledger key carried over from an earlier round.
