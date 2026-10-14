---
round: 1
ledger:
  - key: card6-validate-discussion-commit-unbounded-intent
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card7-postcommit-parse-failure-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card9-darndescribespec-doc-comment-unmentioned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card13-perch-test-repeats-covered-behavior
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings (F1 to F4), in that order in the ledger.
The one MEDIUM is a scope-class gap in card 6's Intent; the one design-class finding is LOW.
None gates, so the loop converges.
