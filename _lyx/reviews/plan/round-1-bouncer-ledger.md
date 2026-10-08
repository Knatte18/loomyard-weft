---
round: 1
ledger:
  - key: card3-duplicated-board-config-fixture
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: card3-done-absent-rows-repeat-setrunstatus
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card1-every-write-rows-repeat-existing-tests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: claim-survives-topology-add-failure-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card1-mutating-method-list-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised five findings (F1 to F5), in the order of the ledger.
The one gating finding is the BLOCKING design finding on Card 3's test fixture.
There is no earlier ledger, so nothing carries forward.
