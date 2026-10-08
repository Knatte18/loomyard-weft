---
round: 2
ledger:
  - key: reconcile-failure-leaves-config-uncommitted
    rounds: [1]
    status: resolved
  - key: reconcile-refusal-unreachable-and-unreached-branches
    rounds: [1]
    status: resolved
  - key: loom-start-resume-refusal-untested
    rounds: [1]
    status: resolved
  - key: snapshotconfig-dead-nil-case
    rounds: [1]
    status: resolved
  - key: snapshotconfig-unreadable-file-treated-absent
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: restore-after-landed-commit-leaves-records-dirty
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: reconcilehubwide-restore-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

All four round 1 keys are resolved: the failed-walk restore, the `reconcileRefusal` branches, the loom verb refusal test and the dead nil case were each fixed and confirmed by the round 2 review.

Round 2 raised three new findings, all in `internal/hubreconcile`.
The MEDIUM design one is that `snapshotConfig` records any read error as an absent file, so the restore deletes an unreadable config file.
The two LOW ones are a restore that runs after a weft commit already landed, and a missing test for the hub-wide restore.
