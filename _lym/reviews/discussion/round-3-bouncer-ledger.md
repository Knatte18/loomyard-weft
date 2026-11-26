---
round: 3
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
    status: resolved
  - key: shed-write-sequence-drops-locked-merge
    rounds: [2]
    status: resolved
  - key: installer-clear-test-only-code
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: stencil-judge-seam-without-consumer
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised gaps in the module cut, the core-and-loop split, the test boundary, the part ordering, the speed target and ref-store behaviour, plus two parent questions.
Round 2 found all of them resolved, with the parent's added entries answering both questions,
and opened two BLOCKING defects in D13: seams unmet in test binaries outside fabric, and a shed-write sequence that lost `internal/state`'s locked merge.

Round 3 finds both resolved and opens two new BLOCKING defects in the D13 fix text:

- The installer's clear is a test-only function in a production package that fabric's tests cannot import.
- The required stencil ordering judge has no fabric consumer, and fabric's second `stencilstore` use has no seam or home.

Each round since the first has found its new defects in the previous round's fix text.

The round 3 review also notes, without filing them, points below its bar.
Among them, `## Open risks` still carries the lines both added entries told the writer to drop, and `support-log.md` still marks Q12 and Q15 open, as already noted in round 2.
