---
round: 1
ledger:
  - key: d2-reader-dollar-expansion-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d1-no-cutoff-run-plan-not-listed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: d1-no-cutoff-untimestamped-transcript
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: support-log-question-ledger-order
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 maps F1 to the D2 dollar-expansion key, F2 to the plan-selection key, F3 to the untimestamped-transcript key and F4 to the Q7 ordering key.
F1 gates; the rest do not.
