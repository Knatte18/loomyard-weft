---
round: 3
ledger:
  - key: fabricengine-test-budget-not-raised
    rounds: [1]
    status: resolved
  - key: renames-typed-edit-not-rename
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: card3-prime-push-test-pinned
    rounds: [2]
    status: resolved
  - key: d1-d2-falsified-docs-uncovered
    rounds: [2]
    status: resolved
  - key: card3-test-budget-no-plan
    rounds: [2]
    status: resolved
  - key: prime-resolution-failure-mechanism-unnamed
    rounds: [2]
    status: resolved
  - key: card5-lists-card7-file
    rounds: [2]
    status: resolved
  - key: card10-omits-falsified-comment
    rounds: [2]
    status: resolved
  - key: card3-anchor-path-breaks-four-tests
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
  - key: card3-both-sides-statements-untouched
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: card2-dangling-deleted-test-reference
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: card2-token-carrying-kinds-doc-missed
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 opened nine findings and round 2 opened eight; round 3 raised none of the round-2 keys again, so all of them are resolved, including the Rename-versus-Edit typing finding that had persisted through rounds 1 and 2.
Round 3 raised four new keys, in order F1 to F4.
`card3-anchor-path-breaks-four-tests` (F1, BLOCKING): Card 3 resolves the prime through `PrimeName(l)` from the anchor path, which fails in the code worktree when the anchor subdirectory exists only on the records side, and breaks `TestCommit_NestedRelPath_ClassifiesWeftFileUnderRelPath` and the `TestFabricSync_CommitsAtEveryRelPathDepth` rows.
`card3-both-sides-statements-untouched` (F2) and the two Card 2 documentation findings (F3, F4) are LOW.
No round-3 finding is `design`, and none sits on a key open in an earlier round.
