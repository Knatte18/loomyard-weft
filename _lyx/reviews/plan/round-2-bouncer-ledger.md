---
round: 2
ledger:
  - key: fabricengine-test-budget-not-raised
    rounds: [1]
    status: resolved
  - key: renames-typed-edit-not-rename
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: card7-footprint-misses-spawn-mutation
    rounds: [1]
    status: resolved
  - key: card2-existing-rollback-tests-unaccounted
    rounds: [1]
    status: resolved
  - key: skippush-coverage-row-unbuildable
    rounds: [1]
    status: resolved
  - key: card3-sync-pair-arm-unpinned
    rounds: [1]
    status: resolved
  - key: card4-5-edit-lists-omit-created-test-files
    rounds: [1]
    status: resolved
  - key: card10-duplicate-fixture-row
    rounds: [1]
    status: resolved
  - key: constraints-carried-by-no-card
    rounds: [1]
    status: resolved
  - key: card6-misses-refusal-assertion
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
  - key: card3-prime-push-test-pinned
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
  - key: d1-d2-falsified-docs-uncovered
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card3-test-budget-no-plan
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: prime-resolution-failure-mechanism-unnamed
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card5-lists-card7-file
    rounds: [2]
    status: open
    class: scope
    severity: NIT
  - key: card10-omits-falsified-comment
    rounds: [2]
    status: open
    class: scope
    severity: NIT
---

Round 1 opened nine findings; the round-2 review did not raise eight of them again, so they are resolved.
Round 2 treats the budget row afresh: Cards 1 and 2 now raise it, and the remaining gap is Card 3's new cases (`card3-test-budget-no-plan`).
The Rename-versus-Edit typing of Cards 4 and 5 (F4) is the one finding that persists, now narrowed to the renames whose declaring files no other card targets whole.
Round 2's two BLOCKING findings are F1 (`card6-misses-refusal-assertion`) and F2 (`card3-prime-push-test-pinned`).
The other round-2 findings map to the new keys in the order F3, F5, F6, F7, F8.
