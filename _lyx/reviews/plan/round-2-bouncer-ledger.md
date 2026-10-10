---
round: 2
ledger:
  - key: card5-gc-failure-subtest-unreachable
    rounds: [1]
    status: resolved
  - key: card2-regression-rows-share-stale-fixture
    rounds: [1]
    status: resolved
  - key: card4-kill-logging-constraint-uncarried
    rounds: [1]
    status: resolved
  - key: card4-pull-row-contradicts-existing-row
    rounds: [1]
    status: resolved
  - key: card1-fixture-deleted-commit-unspecified
    rounds: [1]
    status: resolved
  - key: card4-liveness-check-zombie-race
    rounds: [1]
    status: resolved
  - key: card3-verifyrun-doc-stale
    rounds: [1]
    status: resolved
  - key: card2-forcepackindexfreeze-doc-stale
    rounds: [1]
    status: resolved
  - key: card2-headcontains-notfound-swallows-walk-failure
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card5-remove-subtest-vacuous
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card1-bound-step-needs-own-repository
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card5-pattern-entry-missing-carveout
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eight findings, none of which recurs in round 2.
Round 2 raised four new findings, F1 through F4 in the order listed, none design-class and none BLOCKING.
