---
round: 1
ledger:
  - key: card4-before-card5-breaks-hubforge-tests
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: hub-wide-seeding-wider-than-d4
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: write-then-validate-no-restore-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: concurrent-edit-refusal-no-message-no-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: configsync-doc-statements-disagree
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: docs-still-place-board-yaml-per-worktree
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised six findings, none recurring.
The two gating ones are the card order (the retire card breaks tests that the readers card fixes later) and the missing bound on a failed restore in the hub-wide write.
The rest are a seeding scope wider than the decision licenses, a concurrent-edit refusal without message or test, and two doc inconsistencies.
