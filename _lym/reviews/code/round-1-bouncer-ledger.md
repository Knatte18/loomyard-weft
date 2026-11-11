---
round: 1
ledger:
  - key: preun-skip-parent-review-group-untested
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: findings-file-fields-not-shown-in-report
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: caller-guard-parent-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: findings-file-error-shapes-no-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unknown-state-ask-unmoved-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: await-wait-cut-short-no-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: racing-test-needs-testtiming-keep
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: answer-refusal-ahead-of-failed-write
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, none recurring, since no earlier ledger exists.
Two are gating: the findings-file mapping (F2) and the caller guard's parent branches (F3), both untested design gaps at MEDIUM.
The rest are test-coverage gaps, one keep marker, and one error-ordering fix in `AnswerParentAsk`.
Every fix is local and its result is fixed by the finding text.
