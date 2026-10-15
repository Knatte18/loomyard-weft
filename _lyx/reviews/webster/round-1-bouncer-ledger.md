---
round: 1
ledger:
  - key: rubric-resign-arrow-bullet-merged
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: plan-stencil-duplicate-test-file-rules
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
F1 (`rubric-resign-arrow-bullet-merged`): the plan-review rubric lost the newline before the re-sign arrow bullet, so it renders inside the Member glyphs bullet.
F2 (`plan-stencil-duplicate-test-file-rules`): the plan stencil states the new-test-file and new-test-function rules twice.
Both fixes are local and spelled out by the finding text.
