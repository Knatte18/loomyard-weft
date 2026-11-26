---
round: 1
ledger:
  - key: fabriccli-breaks-import-cut
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: shed-record-writes-cross-cut
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: workspace-bytes-before-verb
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: teardown-child-job-fields
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: retry-outcomes-backoff-placement
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: midlist-failure-and-rollback
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: recorded-mutations-change-output
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: result-types-cannot-move-part-one
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: retarget-gate-scope-overclaim
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: kit-exception-admits-boardkit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reftable-ref-reads-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unset-seam-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: speed-target-baseline-protocol
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: config-values-in-core-tests
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: pattern-text-binds-before-met
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: closed-import-set-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: background-pusher-not-fabric-child
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parent-call-push-retry-dependency
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: parent-call-hubforge-split
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
---

Round 1 is the first round; every key is new and open.

The widest gaps are in the module cut: which packages count as fabric, how shed's record writes and the workspace regeneration cross the cut, and what an unset seam does.
The core-and-loop split leaves retries, mid-list failures, rollback and the teardown child's job undecided.
The rest concern the test boundary, the part ordering, the speed target and unpinned ref-store behaviour.
Two questions wait on the parent: the `push-retry-not-halt` dependency edge and the `internal/hubforge` split.
