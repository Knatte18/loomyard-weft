---
round: 2
ledger:
  - key: preun-skip-parent-review-group-untested
    rounds: [1]
    status: resolved
  - key: findings-file-fields-not-shown-in-report
    rounds: [1]
    status: resolved
  - key: caller-guard-parent-branches-untested
    rounds: [1]
    status: resolved
  - key: findings-file-error-shapes-no-rows
    rounds: [1]
    status: resolved
  - key: unknown-state-ask-unmoved-untested
    rounds: [1]
    status: resolved
  - key: await-wait-cut-short-no-row
    rounds: [1]
    status: resolved
  - key: racing-test-needs-testtiming-keep
    rounds: [1]
    status: resolved
  - key: answer-refusal-ahead-of-failed-write
    rounds: [1]
    status: resolved
  - key: failed-write-ordering-untested
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, two of them gating (the findings-file mapping and the caller guard's parent branches).
Round 2 reports none of them again, so all eight are resolved.
Round 2 raised one new LOW finding: the failed-record-write ordering in `AnswerParentAsk` has no test.
Its fix is local and fixed by the finding text.
