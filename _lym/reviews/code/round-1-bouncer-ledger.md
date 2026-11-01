---
round: 1
ledger:
  - key: seam-test-unlisted-counterpart-rule-dropped
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: bound-notes-cut-from-checks-spec
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rubric-impactsummary-excludes-extends
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: plan-fixer-not-pointed-at-csharp-spec
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four LOW findings, F1 to F4 in that order of the keys above.
None is gating, so the loop converges.
Each fix is a local edit whose text the finding already states.
