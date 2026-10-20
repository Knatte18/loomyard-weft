---
round: 3
ledger:
  - key: generation-under-way-never-judged
    rounds: [1]
    status: resolved
  - key: code-identity-includes-run-records
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: carry-over-changes-discussion-subject
    rounds: [1]
    status: resolved
  - key: amendment-inherits-spent-count
    rounds: [1, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: interrupted-settle-goto-unbounded
    rounds: [1]
    status: resolved
  - key: settled-mark-outlives-generation
    rounds: [1]
    status: resolved
  - key: amendment-hard-cap-accept-only
    rounds: [1]
    status: resolved
  - key: handover-resumes-budget-exhausted-halt
    rounds: [1]
    status: resolved
  - key: driver-rule-misses-status-stop
    rounds: [1]
    status: resolved
  - key: verify-retry-bypass-unbounded
    rounds: [1, 2]
    status: resolved
  - key: merge-review-amendment-unbounded
    rounds: [1]
    status: resolved
  - key: pause-label-names-no-operator
    rounds: [1]
    status: resolved
  - key: relaunch-unchanged-write-clears-pause-reason
    rounds: [1]
    status: resolved
  - key: verdict-validation-misses-seam
    rounds: [1]
    status: resolved
  - key: friction-timeout-min-fate-open
    rounds: [1]
    status: resolved
  - key: d9-trigger-set-mismatch
    rounds: [1]
    status: resolved
  - key: d3-nothing-else-grants-budget
    rounds: [1]
    status: resolved
  - key: d3-reroute-capped-misstated
    rounds: [1]
    status: resolved
  - key: pr-gate-reject-criterion-omits-exempt
    rounds: [1]
    status: resolved
  - key: carry-over-idempotence-not-required
    rounds: [1]
    status: resolved
  - key: d5-read-order-vs-skip-seam
    rounds: [1]
    status: resolved
  - key: merge-review-generation-leaves-settle-record-live
    rounds: [2]
    status: resolved
  - key: amendment-seed-off-by-one
    rounds: [2]
    status: resolved
  - key: amendment-off-switch-unbounded
    rounds: [2]
    status: resolved
  - key: code-base-reads-differ-from-identity
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: document-base-predates-artifact
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-in-test-misreads-commits
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: patch-id-skip-unbounded
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: support-log-stale-against-round-2
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 21 findings, round 2 raised 5, and round 3 raised 7, with no BLOCKING finding in round 3.
The round 3 focus verdicts close the four keys open in round 2 other than the identity key: `verify-retry-bypass-unbounded`, `merge-review-generation-leaves-settle-record-live`, `amendment-seed-off-by-one` and `amendment-off-switch-unbounded`.

`amendment-inherits-spent-count` is reopened by F1.
Shed's budget comparison runs over the history read before the amendment seed is recorded, so a seed over a spent episode is blocked as budget exhausted.
This is a gating finding on a reopened key.

`code-identity-includes-run-records` stays open at LOW through F4: D5's untracked clause has no exclusion set, and a harness file in the worktree can make an unchanged subject read as changed.

New keys: F2 is `code-base-reads-differ-from-identity`, F3 is `document-base-predates-artifact`, F5 is `merge-in-test-misreads-commits`, F6 is `patch-id-skip-unbounded` and F7 is `support-log-stale-against-round-2`.
Review to key mapping: F1 to `amendment-inherits-spent-count`, F2 to `code-base-reads-differ-from-identity`, F3 to `document-base-predates-artifact`, F4 to `code-identity-includes-run-records`, F5 to `merge-in-test-misreads-commits`, F6 to `patch-id-skip-unbounded`, F7 to `support-log-stale-against-round-2`.
