---
round: 1
ledger:
  - key: fabricengine-test-budget-not-raised
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: renames-typed-edit-not-rename
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card7-footprint-misses-spawn-mutation
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card2-existing-rollback-tests-unaccounted
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: skippush-coverage-row-unbuildable
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card3-sync-pair-arm-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card4-5-edit-lists-omit-created-test-files
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card10-duplicate-fixture-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: constraints-carried-by-no-card
    rounds: [1]
    status: open
    class: scope
    severity: NIT
---

Round 1 is the first round, so there is no prior ledger.
It opened nine findings: F1 to F9 in the order of the keys above.
The BLOCKING one is the `internal/fabricengine` test budget row, which no card raises.
The one gating-class finding is the `Rename` versus `Edit` typing of Cards 4 and 5.
