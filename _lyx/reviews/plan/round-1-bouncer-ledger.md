---
round: 1
ledger:
  - key: card2-unquoted-glob-outside-admits
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card8-awaited-shell-row-fails
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: card2-duplicate-classifier-rows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card4-budget-premise-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card6-impactsummary-call-site-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised five findings (F1 to F5), all open.
Card 2 has two (the unbounded glob paths and the duplicated rows), Card 8 has the failing awaited-shell row, and Cards 4 and 6 have consistency slips.
