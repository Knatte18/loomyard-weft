---
round: 4
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
  - key: pattern-md-over-size-cap-after-merge
    rounds: [3]
    status: resolved
---

Round 1 raised five findings, all fixed and confirmed resolved in round 2.
Round 3 raised one BLOCKING finding: the merge-in left `PATTERN.md` over the size cap.
Round 4 confirmed the tightening fixed it, with no rule lost from the touched entries, and found nothing else.
