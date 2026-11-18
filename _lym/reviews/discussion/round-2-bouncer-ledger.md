---
round: 2
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
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
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: protected-and-view-push-failure-run-fate-unstated
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: prime-guard-and-branch-record-depend-on-lym
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: readers-outside-fabric-read-marker-from-git
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised many design findings; round 2 resolved all but two keys.
`d9-misses-webster-reconcile-head` stays open as F29: the walk admits records-only commits but still refuses D5's own strip merge-ins.
`build-now-or-only-for-rewrite` was a parent decision in round 1; added entries settled it, but the record's Open risks and the support log's Q6 still read it as open (F28), so it is now a mechanical consistency fix.
Four new BLOCKING design keys arose where the fixes met existing code:
the transition seam's mid-merge probe reads the shared worktree (F24),
the run's fate on a protected or failed view push is unstated (F25),
the prime guard rests on `.lym` state against `PATTERN-lym-files` (F26),
and readers outside fabric need records paths that only git can give (F27).
