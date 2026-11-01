---
round: 1
ledger:
  - key: spec-gate-vocabulary-omits-clean
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: clean-gate-abs-root-refusal-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: tapestry-code-burler-fix-clean-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings, all local corrections.
F1 maps to `spec-gate-vocabulary-omits-clean`, F2 to `clean-gate-abs-root-refusal-untested`, F3 to `tapestry-code-burler-fix-clean-unpinned`.
None is gating, so the loop converged in round 1.
