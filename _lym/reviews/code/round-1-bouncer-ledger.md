---
round: 1
ledger:
  - key: issue-102-asks-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: keeper-signal-survival-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: keeper-flag-arming-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: non-spawner-waiter-reads-exit-not-recorded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: loopspec-spawn-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: onstart-contract-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: keeper-start-failure-and-exit-code-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: status-moved-and-empty-producer-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: now-reads-clause-moved-history-only
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
Gating: the two untested keeper behaviours, signal survival and the keeper flag on the arming-refusal path.
The omitted issue #102 asks are deferred to the run's parent.
The remaining findings are doc corrections, test gaps and one wording clause in `deadLoopStop`.
The non-spawner waiter finding is marked `design-wrong` because its fix chooses between a bounded re-read and a keeper-liveness check.
