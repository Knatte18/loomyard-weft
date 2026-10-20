---
round: 1
ledger:
  - key: generation-under-way-never-judged
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: code-identity-includes-run-records
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: carry-over-changes-discussion-subject
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: amendment-inherits-spent-count
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: interrupted-settle-goto-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: settled-mark-outlives-generation
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: amendment-hard-cap-accept-only
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: handover-resumes-budget-exhausted-halt
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: driver-rule-misses-status-stop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: verify-retry-bypass-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-review-amendment-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pause-label-names-no-operator
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: relaunch-unchanged-write-clears-pause-reason
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: verdict-validation-misses-seam
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: friction-timeout-min-fate-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: d9-trigger-set-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-nothing-else-grants-budget
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-reroute-capped-misstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pr-gate-reject-criterion-omits-exempt
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: carry-over-idempotence-not-required
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d5-read-order-vs-skip-seam
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
The four BLOCKING findings concern D5's settle-record comparison and D1/D6's episode counting: a generation under way is never judged, the code and Discussion identities include the run's own records, and an amendment inherits the spent count.
The MEDIUM design findings are missing bounds on gotos, the hard cap, the hand-over, the verify-retry bypass, Merge-Review amendments and the pause label.
The LOW and NIT findings are local corrections and wording mismatches against the existing code.
Key to review mapping, F1 to F21 in order, one key each.
