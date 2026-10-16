---
round: 2
ledger:
  - key: d5-bound-omits-compile-only-skip
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-rubric-scope-vs-rework-rule
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d1-stop-and-report-unscoped
    rounds: [1]
    status: resolved
  - key: implementer-empty-list-fallback-vs-shell-guard
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: compile-only-rule-vs-rendered-gate-steps
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: empty-list-render-unstated-plan-implementer
    rounds: [1]
    status: resolved
  - key: acceptance-omits-d1-rubric-and-no-parent
    rounds: [1]
    status: resolved
  - key: d5-rationale-carries-deliberation
    rounds: [1]
    status: resolved
---

Round 2 raised four findings and no BLOCKING one.
Two gating-class MEDIUM findings reopen round-1 keys: the D5 Bound now misses the landing consumer of the verify section, and the implementer's dropped `tmux`/`llm` compile-only sentence leaves the empty-list fallback unsafe.
The D1 rework-stencil finding and the build-tagged-clause finding continue as non-gating consistency keys.
Four round-1 keys were not raised again and are resolved.
