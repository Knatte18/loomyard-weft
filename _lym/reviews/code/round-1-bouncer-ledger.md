---
round: 1
ledger:
  - key: ancestry-query-argument-order-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: ancestry-query-error-path-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: session-ref-fallback-forms-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: changed-branch-fix-review-halt-session-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: delete-push-trace-line-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised only test-coverage gaps on the diff's new behavior.
The review reports every goal point of the answer key as built and no PATTERN entry broken.
The one gating item is the unpinned argument order of the ancestry query in the fixer loop.
