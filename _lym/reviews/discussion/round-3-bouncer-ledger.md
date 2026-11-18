---
round: 3
ledger:
  - key: d8-publish-not-idempotent
    rounds: [1]
    status: resolved
  - key: local-records-commits-parent-ahead
    rounds: [1]
    status: resolved
  - key: protected-prime-branch-unbounded
    rounds: [1]
    status: resolved
  - key: d7-predicates-undefined-forgeable
    rounds: [1]
    status: resolved
  - key: d6-code-tip-handover-on-records-branch
    rounds: [1]
    status: resolved
  - key: pr-head-sites-take-task-head
    rounds: [1]
    status: resolved
  - key: empty-records-commit-refuses-with-staged-code
    rounds: [1]
    status: resolved
  - key: d9-misses-webster-reconcile-head
    rounds: [1, 2]
    status: resolved
  - key: records-paths-from-one-marker
    rounds: [1]
    status: resolved
  - key: merge-write-between-steps-2-and-3-lost
    rounds: [1]
    status: resolved
  - key: add-inherits-parent-run-records
    rounds: [1]
    status: resolved
  - key: prime-records-follow-operator-checkout
    rounds: [1]
    status: resolved
  - key: held-work-note-never-reaches-comparison
    rounds: [1]
    status: resolved
  - key: single-repo-flag-half-made-origin
    rounds: [1]
    status: resolved
  - key: tool-facts-carry-no-pin
    rounds: [1]
    status: resolved
  - key: build-now-or-only-for-rewrite
    rounds: [1, 2]
    status: resolved
  - key: pattern-cwd-resolution-overview-weft-root
    rounds: [1]
    status: resolved
  - key: replay-rewrites-warp-sha-over-foreign-code
    rounds: [1]
    status: resolved
  - key: d7-any-branch-scope-covers-lym-bolt
    rounds: [1]
    status: resolved
  - key: owed-prime-push-names-no-failed-push-class
    rounds: [1]
    status: resolved
  - key: view-base-not-tied-to-pr-base
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-disagree-with-log
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-miss-stated-behaviours
    rounds: [1]
    status: resolved
  - key: transition-seam-mid-merge-probe-reads-code-merge
    rounds: [2]
    status: resolved
  - key: protected-and-view-push-failure-run-fate-unstated
    rounds: [2]
    status: resolved
  - key: prime-guard-and-branch-record-depend-on-lym
    rounds: [2]
    status: resolved
  - key: readers-outside-fabric-read-marker-from-git
    rounds: [2]
    status: resolved
  - key: records-commit-abort-resets-live-records
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: merge-verb-rerun-abort-removes-in-progress-guard
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised many design findings; round 2 resolved all but two keys and added four.
Round 3 resolved every one of those (F24 to F29).
Two new BLOCKING design keys arose in the crash-recovery text those fixes added.
`records-commit-abort-resets-live-records` (F30): D4's abort of a merge that stopped before D5 step 3 hard-resets the worktree's live records, so the commit lands empty and the run acts past a lost transition.
`merge-verb-rerun-abort-removes-in-progress-guard` (F31): D5's rule that a merge verb re-run aborts any merge in progress removes the already-in-progress guard and contradicts D4's parked-merge handling.
