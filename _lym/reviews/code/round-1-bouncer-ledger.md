---
round: 1
ledger:
  - key: sampler-wake-path-done-flag-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: sampler-record-read-error-refuses-parent-orch
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
F1 maps to `sampler-wake-path-done-flag-untested`: the wake-path done-flag branches (dirty checkout resumed and mailed, damaged record Stuck) have no test row.
F2 maps to `sampler-record-read-error-refuses-parent-orch`: an unreadable sampler record refuses the run's parent orch in `decisionVerb`.
Both fixes are local and the finding text fixes their result.
