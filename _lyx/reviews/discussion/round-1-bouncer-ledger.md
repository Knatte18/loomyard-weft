---
round: 1
ledger:
  - key: d3-bound-false-premise-operator-notes
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: goal-vs-d1-zero-bound-fallback
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d2-way-forward-names-no-verb
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: title-form-second-copy-render-instead
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: open-risk-misses-other-bound-callers
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: leaf-packages-constraint-binds-nothing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

First round, no prior ledger.
F1 (D3 bound misses operator-written decision notes without the title) and F4 (render the title form from `internal/alignment` rather than copy it into the orch stencil) are the gating design findings.
F2, F3, F5 and F6 are local wording and Constraints corrections the review's fix text already fixes.
