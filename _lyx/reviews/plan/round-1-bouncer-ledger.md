---
round: 1
ledger:
  - key: tighter-read-rules-fixtures-untargeted
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: usage-entry-dropped-on-entry-time-attach
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: clean-converged-row-duplicates-verdicts-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: construction-refusal-outside-failure-table
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 (F1 to F4) mapped to the keys above, in order.
F1: cards 5 and 6 tighten reads (review parse, review half of the usage record) without targeting the fixtures that now fail.
F2: card 7 omits usage entries for passes harvested by `awaitLiveJudge` and `awaitLiveSeed`.
F3 and F4 are test-economy duplicates and misplacements.
