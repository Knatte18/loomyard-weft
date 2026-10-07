---
round: 4
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
    status: resolved
  - key: card3-both-sides-statements-untouched
    rounds: [3, 4]
    status: open
    class: scope
    severity: MEDIUM
  - key: card2-dangling-deleted-test-reference
    rounds: [3]
    status: resolved
  - key: card2-token-carrying-kinds-doc-missed
    rounds: [3]
    status: resolved
  - key: card2-adopted-test-prefix-comment
    rounds: [4]
    status: open
    class: scope
    severity: LOW
---

Rounds 1 to 3 opened and closed the early keys, including the Rename-versus-Edit typing finding that persisted through rounds 1 and 2.
Round 3's BLOCKING anchor-path finding (`card3-anchor-path-breaks-four-tests`) is not raised again, and round 4 verified every anchor-subdirectory fixture, so it is resolved.
The two Card 2 documentation keys from round 3 are likewise verified as named, so they are resolved.
`card3-both-sides-statements-untouched` reopened in round 4 as F1 (MEDIUM, scope): Card 3 still omits `fabriccli/spawn.go` and the `pushanchored.go` header and `PushPairAnchored` doc.
`card2-adopted-test-prefix-comment` is new in round 4 as F2 (LOW, scope): the empty-prefix comment in `TestAddRollback_AdoptedWeftBranchSurvives`.
No round-4 finding is `design`, so the loop converged.
