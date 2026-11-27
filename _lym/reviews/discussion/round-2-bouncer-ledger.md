---
round: 2
ledger:
  - key: local-landing-origin-unreachable
    rounds: [1]
    status: resolved
  - key: second-landing-parent-ahead
    rounds: [1]
    status: resolved
  - key: board-done-not-ordered-after-parent
    rounds: [1]
    status: resolved
  - key: refused-target-reads-as-pushed
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: wait-record-names-run-lock
    rounds: [1]
    status: resolved
  - key: merge-to-owed-record-crash-gap
    rounds: [1]
    status: resolved
  - key: records-commit-before-parent-push
    rounds: [1]
    status: resolved
  - key: owes-landing-push-undefined
    rounds: [1]
    status: resolved
  - key: waiting-run-never-summarised
    rounds: [1]
    status: resolved
  - key: teardown-with-pushes-off
    rounds: [1]
    status: resolved
  - key: push-path-sorting-rule
    rounds: [1]
    status: resolved
  - key: interval-undercut-by-retry-loops
    rounds: [1]
    status: resolved
  - key: halt-note-written-by-step
    rounds: [1]
    status: resolved
  - key: two-pushers-on-beam-branch
    rounds: [1]
    status: resolved
  - key: ignore-rules-from-tree-unpinned
    rounds: [1]
    status: resolved
  - key: removal-base-and-resume-placement
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reachability-bound-unpinned-no-way-forward
    rounds: [1]
    status: resolved
  - key: pattern-pair-remote-text-vs-sync-pushes
    rounds: [1]
    status: resolved
  - key: issue-238-no-red-first-test
    rounds: [1]
    status: resolved
  - key: permanent-faults-read-as-network
    rounds: [1]
    status: resolved
  - key: auth-classification-stated-as-new
    rounds: [1]
    status: resolved
  - key: other-origin-moving-paths-unsettled
    rounds: [1]
    status: resolved
  - key: catchup-prime-push-pusher
    rounds: [1]
    status: resolved
  - key: hand-commit-reproduction-incomplete
    rounds: [1]
    status: resolved
  - key: removal-bound-omits-ignored-secrets
    rounds: [1]
    status: resolved
  - key: open-risk-writer-choice
    rounds: [1]
    status: resolved
  - key: third-config-key-undeclared
    rounds: [1]
    status: resolved
  - key: land-row-stops-on-owed-landing-push
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: one-landing-entry-for-several-runs
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: records-hold-bypassed-by-other-pushes
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: parent-pusher-no-owner-verdict
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 2 resolved every round 1 key but two.
`refused-target-reads-as-pushed` stays open: the record says clearing the refused-push record restarts the pusher,
but names no writer that clears the mark, and stale owed entries have no remover either.
`removal-base-and-resume-placement` stays open: the removal's base now comes from a merge record
that is written only after the clean check the removal must precede.

The four new keys are all BLOCKING and all follow from handing the parent push to the parent target's pusher:
batten's Land row after a local landing, several landed runs sharing one landing entry,
other pushes bypassing the records hold, and the missing owner-machine verdict.

The round 2 review also lists lower findings it did not raise under its incremental rule.
They have no ledger key.
