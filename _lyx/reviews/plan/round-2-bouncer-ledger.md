---
round: 2
ledger:
  - key: card3-duplicated-board-config-fixture
    rounds: [1]
    status: resolved
  - key: card3-done-absent-rows-repeat-setrunstatus
    rounds: [1]
    status: resolved
  - key: card1-every-write-rows-repeat-existing-tests
    rounds: [1]
    status: resolved
  - key: claim-survives-topology-add-failure-untested
    rounds: [1]
    status: resolved
  - key: card1-mutating-method-list-incomplete
    rounds: [1]
    status: resolved
  - key: setrunstatus-locked-entry-row-repeats-runlock
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card3-no-board-config-row-wrong-error-no-warning-assert
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: markdonefixturedir-keeps-openhub-body-copy
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised five findings, one BLOCKING design finding and four lower ones.
None was raised again in round 2, so all five are resolved.
Round 2 raised three new consistency findings: a re-pinned locked-entry row in Card 2's `TestSetRunStatus`, a mis-described error and unasserted warning in Card 3's no-board-config row, and a leftover copy of the board-opening body in `markDoneFixtureDir`.
