---
round: 1
ledger:
  - key: merge-in-pr-review-row-and-describe-still-run
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: merge-task-body-missing-space
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: normalizetask-doc-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: test-row-claims-unasserted-rule-read-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings and no earlier ledger exists.
F1 is a decision finding deferred to the run's parent.
F2 and F3 are local text corrections.
F4 is a test row whose name claims an assertion it lacks.
