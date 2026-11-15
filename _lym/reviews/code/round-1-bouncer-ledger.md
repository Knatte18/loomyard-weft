---
round: 1
ledger:
  - key: seed-only-reconcile-row-for-code-reviewer-model
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: contained-tests-lack-testtiming-keep
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger to carry forward.
Both findings are LOW and local: add two reconcile rows for the new model key (F1), and add `//testtiming:keep` markers to `TestTemplateLadder` and `TestTemplateDocsProfile` (F2).
