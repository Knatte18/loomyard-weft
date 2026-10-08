---
round: 1
ledger:
  - key: segment-color-decisions-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: dead-removers-deleted-late
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: open-hedges-in-test-targets
    rounds: [1]
    status: open
    class: decision
    severity: NIT
---

Round 1 is the first round, so there is no earlier history.
F1 maps to `segment-color-decisions-uncarried`: the plan has no card routing start's color play and `Runner.TypeColor` through the verified send.
F2 maps to `dead-removers-deleted-late`: removers orphaned by two cards are deleted only by a later one, and `PATTERN-shuttle-stop` lands before they are gone.
F3 maps to `open-hedges-in-test-targets`: three cards leave a choice of test open.
