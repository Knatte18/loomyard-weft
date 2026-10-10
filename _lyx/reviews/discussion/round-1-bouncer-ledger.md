---
round: 1
ledger:
  - key: seed-signature-matches-stencil-sync
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d5-drop-refused-with-board-commit-ahead
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d1-ancestry-unanswerable-outside-loomyard-and-specs
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: stamp-retry-cannot-clear-hubwide-config-commit
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: ancestry-read-mechanism-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d9-bound-rests-on-false-d8-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: failing-probe-silently-stops-gc
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: acceptance-criteria-miss-behaviours
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: unresolvable-terms-for-plan-producer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

First round, no prior ledger.
Three BLOCKING design findings concern D5's drop signature and primitive and D1's ancestry source.
The remaining findings are consistency gaps between D1, D4, D5, D8, D9 and the acceptance criteria, plus one LOW design gap in D8's failing probe.
