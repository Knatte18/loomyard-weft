---
round: 1
ledger:
  - key: card5-gc-failure-subtest-unreachable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: card2-regression-rows-share-stale-fixture
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card4-kill-logging-constraint-uncarried
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: card4-pull-row-contradicts-existing-row
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card1-fixture-deleted-commit-unspecified
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card4-liveness-check-zombie-race
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card3-verifyrun-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card2-forcepackindexfreeze-doc-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised eight findings, none recurring.
Keys map to F1 through F8 in the order listed.
The BLOCKING one is card 5's gc-failure subtest, which assumes a held `gc.pid` makes `git gc --auto` fail when it exits 0 silently.
