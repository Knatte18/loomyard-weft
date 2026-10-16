---
round: 1
ledger:
  - key: d5-bound-omits-compile-only-skip
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-rubric-scope-vs-rework-rule
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d1-stop-and-report-unscoped
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: implementer-empty-list-fallback-vs-shell-guard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: compile-only-rule-vs-rendered-gate-steps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: empty-list-render-unstated-plan-implementer
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: acceptance-omits-d1-rubric-and-no-parent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-rationale-carries-deliberation
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, none recurring.
The gating pair is the D5 Bound gap and the implementer fallback that contradicts the kept shell-guard row.
D1's scope and stop-and-report ambiguities are MEDIUM but non-gating.
The remaining four are local corrections.
