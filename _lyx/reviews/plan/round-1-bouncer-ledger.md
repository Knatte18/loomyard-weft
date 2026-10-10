---
round: 1
ledger:
  - key: card11-registry-names-pin-missing
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card6-done-advisor-notifies-chair
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card6-misnamed-seat-strand-left-running
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: cards3-5-seat-channel-bound-unstated
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card5-reserved-marker-refusal-unbuilt
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card3-chair-block-channel-wording
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card6-chair-wait-error-unspecified
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card11-retests-unknown-row-key
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card12-repeats-nested-include-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card5-uses-embedded-seat-block-vars
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: reserved-markers-split-cards-4-5
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eleven findings, none `design` class.
The BLOCKING one is card 11's missing edit to the registry's expected-names test.
The MEDIUM ones are card 6's Done-advisor notice and misnamed-seat strand, the unstated bound on the seat channel across cards 3 and 5, and card 5's planned refusal that no card builds.
The LOW and NIT findings are channel wording, a chair `Wait` error case, three duplicated or misdirected test and `Uses` entries, and the `ReservedMarkers` split.
