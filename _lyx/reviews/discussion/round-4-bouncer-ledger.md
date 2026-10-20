---
round: 4
ledger:
  - key: generation-under-way-never-judged
    rounds: [1]
    status: resolved
  - key: code-identity-includes-run-records
    rounds: [1, 2, 3, 4]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: carry-over-changes-discussion-subject
    rounds: [1]
    status: resolved
  - key: amendment-inherits-spent-count
    rounds: [1, 3]
    status: resolved
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
    status: resolved
  - key: document-base-predates-artifact
    rounds: [3]
    status: resolved
  - key: merge-in-test-misreads-commits
    rounds: [3]
    status: resolved
  - key: patch-id-skip-unbounded
    rounds: [3]
    status: resolved
  - key: support-log-stale-against-round-2
    rounds: [3]
    status: resolved
  - key: d3-goto-after-halt-grants-budget
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unharvested-verdict-read-as-older-binary
    rounds: [4]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised 21 findings, round 2 raised 5, round 3 raised 7 and round 4 raised 3, with no BLOCKING finding since round 2.

Round 4 mapping: F1 to `d3-goto-after-halt-grants-budget`, F2 to `unharvested-verdict-read-as-older-binary`, F3 to `code-identity-includes-run-records`.

Round 4's focus verdicts close the amendment seed over a spent episode, the document segment base and the code segment base, so `amendment-inherits-spent-count`, `document-base-predates-artifact` and `code-base-reads-differ-from-identity` are resolved.
The remaining round 3 keys were not raised again and are resolved.

Open: F1 is MEDIUM design on a key first raised in round 4, carried into Open risks.
F2 and F3 are LOW; F3 is the told skip list lacking a stated bound, the tail of the identity key.
