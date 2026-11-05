---
round: 1
ledger:
  - key: merge-fixer-not-used-for-conflicts
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: real-filing-seams-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: repeated-done-literal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: partial-filing-misleading-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
F1 (`merge-fixer-not-used-for-conflicts`) is an unrecorded parent correction, deferred to the run's parent.
F2 (`real-filing-seams-untested`) is the only gating finding.
F3 and F4 are local corrections whose text fixes the result.
