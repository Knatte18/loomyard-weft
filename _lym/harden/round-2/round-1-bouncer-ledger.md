---
round: 1
ledger:
  - key: seed-board-push-lands-after-rollback
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: faulted-fetch-refspec-read-as-two-repositories
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: prime-key-names-missing-branch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: seed-way-forward-misnames-hook-decline
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seeded-records-branch-has-no-upstream
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open because their fixes are unreviewed text.

- `seed-board-push-lands-after-rollback` is F1.
  A board push that dies without a definite rejection can still be applied by origin's receive-pack after the seed has deleted the records branch and removed the hub.
- `faulted-fetch-refspec-read-as-two-repositories` is F2.
  `originNamespaceAt` reads a records store with no or an identity fetch refspec as two repositories, and a board write then aims at plain `refs/heads/main`.
- `prime-key-names-missing-branch` is F3.
  `primaryWeftBranch` accepts a `lym.primeRecordsBranch` naming no branch, so the sweep lists the real prime records branch as deletable.
- `seed-way-forward-misnames-hook-decline` is F4.
  A hook that declines only the board ref gets the "host refuses refs outside `refs/heads/`" way forward.
- `seeded-records-branch-has-no-upstream` is F5.
  The seed sets an upstream for the board only, so `@{u}` on the seeded records branch fails.

No earlier ledger exists, so nothing is carried forward.
