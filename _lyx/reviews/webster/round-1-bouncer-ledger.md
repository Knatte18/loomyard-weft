---
round: 1
ledger:
  - key: reconcile-failure-leaves-config-uncommitted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: reconcile-refusal-unreachable-and-unreached-branches
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: loom-start-resume-refusal-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: snapshotconfig-dead-nil-case
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first judged round, so there is no earlier ledger.
The gating finding is a failed walk step in `internal/hubreconcile/walk.go` that leaves written config uncommitted, after which a retry stamps the build over it.
The other three are a dead branch plus unreached branches in `reconcileRefusal`, a missing loom-verb test for the new reconcile refusals, and a dead `data == nil` case in `snapshotConfig`.
