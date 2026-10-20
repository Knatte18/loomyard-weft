---
round: 2
ledger:
  - key: generation-under-way-never-judged
    rounds: [1]
    status: resolved
  - key: code-identity-includes-run-records
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: carry-over-changes-discussion-subject
    rounds: [1]
    status: resolved
  - key: amendment-inherits-spent-count
    rounds: [1]
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
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: amendment-seed-off-by-one
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: amendment-off-switch-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised 21 findings; round 2 raised 5, and the fixes closed 19 of the round 1 keys, none re-raised.
Two round 1 keys came back reshaped.
`verify-retry-bypass-unbounded` is F2: the fix keyed the bound on a settle time that a round-by-round re-settle never advances, so the bypass never closes.
`code-identity-includes-run-records` is F4: the identity's form is unstated and its `_lyx` restriction rests on a false premise, since `_lyx` is the weft side.
Three keys are new.
`merge-review-generation-leaves-settle-record-live` is F1 and BLOCKING: Merge-Review's case 2 starts a generation without bumping the generation record, so the kept settle record stays live and re-archives on every Call.
`amendment-seed-off-by-one` is F3 and `amendment-off-switch-unbounded` is F5.
Key to review mapping: F1 to the Merge-Review key, F2 to `verify-retry-bypass-unbounded`, F3 to `amendment-seed-off-by-one`, F4 to `code-identity-includes-run-records`, F5 to `amendment-off-switch-unbounded`.
