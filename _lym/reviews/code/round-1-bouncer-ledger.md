---
round: 1
ledger:
  - key: skillload-test-three-behaviors-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: classifyskillload-doc-stale-outputs
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: printedfile-param-shadows-path-package
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
F1 asks for three test rows folded into `TestClassifySkillLoad`, F2 a one-phrase doc fix, F3 a parameter rename; all three are local corrections.
