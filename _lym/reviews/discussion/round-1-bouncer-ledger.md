---
round: 1
ledger:
  - key: local-landing-origin-unreachable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: second-landing-parent-ahead
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: board-done-not-ordered-after-parent
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: refused-target-reads-as-pushed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: wait-record-names-run-lock
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: merge-to-owed-record-crash-gap
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: records-commit-before-parent-push
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: owes-landing-push-undefined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: waiting-run-never-summarised
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: teardown-with-pushes-off
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: push-path-sorting-rule
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: interval-undercut-by-retry-loops
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: halt-note-written-by-step
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: two-pushers-on-beam-branch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ignore-rules-from-tree-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: removal-base-and-resume-placement
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reachability-bound-unpinned-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pattern-pair-remote-text-vs-sync-pushes
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: issue-238-no-red-first-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: permanent-faults-read-as-network
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: auth-classification-stated-as-new
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: other-origin-moving-paths-unsettled
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: catchup-prime-push-pusher
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hand-commit-reproduction-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: removal-bound-omits-ignored-secrets
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: open-risk-writer-choice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: third-config-key-undeclared
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so every key is new and none is resolved.

The open work clusters in these areas:

- Local landing against an owed or unreachable origin: reachability, a parent ahead of origin, push ordering, crash recovery and the meaning of "owes a landing push".
- The push ledger: refused targets, summary triggers, push targets sharing a branch, and the retry interval.
- The driver's network wait: its lock and its halt note.
- Reset reachability, ignored-file removal, and the sorting of push paths.
