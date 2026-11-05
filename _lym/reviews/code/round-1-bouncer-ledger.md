---
round: 1
ledger:
  - key: reflog-subject-declared-twice
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: failed-record-step-reports-landed-push-failed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: held-lock-test-checks-too-early
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: default-push-remote-choice-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: record-push-no-record-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: outdated-comments-left
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, F1 to F6, in the order of the ledger keys above.
F1 is BLOCKING under `PATTERN-named-constants`.
F2 changes the contract of a failed record step after a landed transfer, so its fix needs a fresh review.
F3 to F6 are local corrections the review's fix text already fixes.
No key predates this round.
