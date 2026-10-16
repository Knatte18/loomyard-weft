---
round: 1
ledger:
  - key: stencil-escalations-stray-blank-line
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: table-test-missing-t-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger to carry forward.
F1 maps to `stencil-escalations-stray-blank-line`, a blank line splitting a bullet list in `contracts/stencils/orch/orch-template-role.md`.
F2 maps to `table-test-missing-t-parallel`, a table test in `internal/orchengine/prompt_test.go` without its own `t.Parallel`.
Both are cosmetic and non-gating, and both fixes follow from the finding text alone.
