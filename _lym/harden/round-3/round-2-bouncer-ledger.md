---
round: 2
ledger:
  - key: seed-board-push-lands-after-rollback
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: faulted-fetch-refspec-read-as-two-repositories
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prime-key-names-missing-branch
    rounds: [1]
    status: resolved
  - key: seed-way-forward-misnames-hook-decline
    rounds: [1]
    status: resolved
  - key: seeded-records-branch-has-no-upstream
    rounds: [1]
    status: resolved
  - key: prime-key-refusal-spells-suffix-literal
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised five findings; round 2 resolved three of them and raised one new finding.

- `seed-board-push-lands-after-rollback` stays open.
  The board push is fixed: a landed board is withdrawn before the restore, and an unlisted one keeps the hub.
  The records push ending without origin's answer is rolled back blind, and a kept-hub path has no test.
  It is F6 in round 2.
- `faulted-fetch-refspec-read-as-two-repositories` stays open at LOW.
  The fix reads the push refspecs and the prime key; a store that keeps only `branch.<b>.merge` still reads as two repositories.
  It is F8 in round 2.
- `prime-key-names-missing-branch` is resolved: a key naming a missing branch refuses with a `git config` way forward.
- `seed-way-forward-misnames-hook-decline` is resolved: a hook declining the board is named as origin's hooks or rules.
- `seeded-records-branch-has-no-upstream` is resolved: the seeded records branch answers `@{u}` and shows its ahead count.
- `prime-key-refusal-spells-suffix-literal` is new, F7.
  The refusal the prime-key fix added spells the records suffix as a literal and its placeholder invites the wrong branch.
  A key naming the board branch or another suffixed branch passes and exposes the prime's records branch to the sweep.
