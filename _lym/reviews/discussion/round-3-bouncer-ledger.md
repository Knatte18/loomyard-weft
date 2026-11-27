---
round: 3
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
    status: resolved
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
    rounds: [1, 2, 3]
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
    status: resolved
  - key: one-landing-entry-for-several-runs
    rounds: [2]
    status: resolved
  - key: records-hold-bypassed-by-other-pushes
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: parent-pusher-no-owner-verdict
    rounds: [2]
    status: resolved
  - key: refused-prime-code-push-way-forward-dead-end
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: parent-pusher-drops-run-owner-check
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised a broad set of keys, and round 2 resolved all of them but `refused-target-reads-as-pushed`
and `removal-base-and-resume-placement`.
Round 2's new keys came from handing the parent push to the parent target's pusher.

Round 3 resolves the refused-mark writer, the Land row, the shared landing entry and the parent's owner verdict.
Still open:

- `removal-base-and-resume-placement`, open in every round: D9's base is persisted after the merge-in,
  so a kill between the merge and the record write loses it.
- `records-hold-bypassed-by-other-pushes`: `lym fabric remove` of a standing pair marks the entry done
  and pushes the held records.
- `refused-prime-code-push-way-forward-dead-end`: `lym fabric push` in the prime does not push the code branch,
  and the diverged and handed-over cases are unsettled.
- `parent-pusher-drops-run-owner-check`: the pusher reads the parent's owner, not the landed run's.

The round 3 review also lists observations on unchanged text that it did not raise under its incremental rule.
They have no ledger key.
