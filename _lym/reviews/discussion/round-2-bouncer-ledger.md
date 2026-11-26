---
round: 2
ledger:
  - key: fabriccli-breaks-import-cut
    rounds: [1]
    status: resolved
  - key: shed-record-writes-cross-cut
    rounds: [1]
    status: resolved
  - key: workspace-bytes-before-verb
    rounds: [1]
    status: resolved
  - key: teardown-child-job-fields
    rounds: [1]
    status: resolved
  - key: retry-outcomes-backoff-placement
    rounds: [1]
    status: resolved
  - key: midlist-failure-and-rollback
    rounds: [1]
    status: resolved
  - key: recorded-mutations-change-output
    rounds: [1]
    status: resolved
  - key: result-types-cannot-move-part-one
    rounds: [1]
    status: resolved
  - key: retarget-gate-scope-overclaim
    rounds: [1]
    status: resolved
  - key: kit-exception-admits-boardkit
    rounds: [1]
    status: resolved
  - key: reftable-ref-reads-unpinned
    rounds: [1]
    status: resolved
  - key: unset-seam-unbounded
    rounds: [1]
    status: resolved
  - key: speed-target-baseline-protocol
    rounds: [1]
    status: resolved
  - key: config-values-in-core-tests
    rounds: [1]
    status: resolved
  - key: pattern-text-binds-before-met
    rounds: [1]
    status: resolved
  - key: closed-import-set-unnamed
    rounds: [1]
    status: resolved
  - key: background-pusher-not-fabric-child
    rounds: [1]
    status: resolved
  - key: parent-call-push-retry-dependency
    rounds: [1]
    status: resolved
  - key: parent-call-hubforge-split
    rounds: [1]
    status: resolved
  - key: seams-missing-in-outside-test-binaries
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: shed-write-sequence-drops-locked-merge
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised gaps in the module cut, the core-and-loop split, the test boundary, the part ordering, the speed target and ref-store behaviour, plus two parent questions.
Round 2 finds every one of them resolved; the parent's added entries answered both questions.

Round 2 opens two new BLOCKING design defects, both in D13 and both arising from the round 1 fixes:

- The required seams and the nil workspace builder are unmet in test binaries outside fabric.
- The shed-write sequence loses `internal/state`'s locked, unknown-key-keeping merge.

The round 2 review also noted, without raising them, defects below BLOCKING.
Among them, Open risks 1 to 3 and Q12 and Q15 still stand although the added entries told the writer to drop or mark them.
