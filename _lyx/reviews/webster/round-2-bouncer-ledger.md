---
round: 2
ledger:
  - key: boot-signal-test-not-discriminating
    rounds: [1]
    status: resolved
  - key: orch-hold-memory-not-forgotten-on-pass
    rounds: [1]
    status: resolved
  - key: orch-hold-stuck-survives-restart
    rounds: [1]
    status: resolved
  - key: idle-cpu-report-unmeasured-stat-tick-claim
    rounds: [1]
    status: resolved
  - key: shuttle-wait-leftover-default-constant
    rounds: [1]
    status: resolved
---

Round 1 raised five findings, each fixed in a later commit.
Round 2 found nothing and re-checked the boot-signal, hold-memory and Stuck-restart fixes against the code; none regressed, so all five keys are resolved.
