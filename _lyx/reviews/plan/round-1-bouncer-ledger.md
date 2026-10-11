---
round: 1
ledger:
  - key: d5-slug-choice-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card12-throttle-test-duplicates
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card5-second-refusal-table
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card1-ttl-resolver-rows-retested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card1-wrong-test-file
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round, with no prior ledger.
Its five findings are F1 to F5 in that order, all non-gating.
F1 asks for a CLI-level test of the slug-driven status choice in card 10.
F2 and F3 are test-economy duplicates in cards 12 and 5.
F4 and F5 are a redundant-row and a wrong-file point in card 1.
