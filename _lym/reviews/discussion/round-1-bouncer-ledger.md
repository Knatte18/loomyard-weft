---
round: 1
ledger:
  - key: d8-publish-not-idempotent
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: local-records-commits-parent-ahead
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: protected-prime-branch-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-predicates-undefined-forgeable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-code-tip-handover-on-records-branch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pr-head-sites-take-task-head
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: empty-records-commit-refuses-with-staged-code
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d9-misses-webster-reconcile-head
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: records-paths-from-one-marker
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-write-between-steps-2-and-3-lost
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: add-inherits-parent-run-records
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: prime-records-follow-operator-checkout
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: held-work-note-never-reaches-comparison
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: single-repo-flag-half-made-origin
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tool-facts-carry-no-pin
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: build-now-or-only-for-rewrite
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: pattern-cwd-resolution-overview-weft-root
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: replay-rewrites-warp-sha-over-foreign-code
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-any-branch-scope-covers-lym-bolt
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: owed-prime-push-names-no-failed-push-class
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: view-base-not-tied-to-pr-base
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: auto-mode-assumptions-disagree-with-log
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-criteria-miss-stated-behaviours
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review of the decision record, with no prior ledger.
Five BLOCKING design findings sit where the one-repository design meets code that already runs:
the pull-request view publish, the parent alignment and fast-forward, protected prime branches, D7's predicates and the hand-over's code-tip paths.
Ten more design findings are MEDIUM; most name a concrete site the Decisions skip.
The one `decision` finding, whether to build now or only spec for the fabricengine rewrite, goes to the run's parent.
