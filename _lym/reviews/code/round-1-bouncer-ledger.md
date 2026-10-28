---
round: 1
ledger:
  - key: hubdobbydeps-tasklimit-wiring-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: tasklimit-per-start-reading-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW test-coverage findings: F1 maps to `hubdobbydeps-tasklimit-wiring-untested`, F2 to `tasklimit-per-start-reading-untested`.
Neither gates, and the loop converged in the first round.
