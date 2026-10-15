---
round: 1
ledger:
  - key: all-runs-sleep-line-unconditional
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: journal-degradation-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW consistency findings.
F1 maps to `all-runs-sleep-line-unconditional`: the All runs section prints the machine-sleep line even when no sleep was subtracted.
F2 maps to `journal-degradation-untested`: the degraded journal paths and their header lines have no test.
Neither is gating, and there was no earlier ledger to carry forward.
