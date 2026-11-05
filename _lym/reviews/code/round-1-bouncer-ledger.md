---
round: 1
ledger:
  - key: fixer-pane-record-holds-quiet-notice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-test-row-for-another-slug-gate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: run-end-stamp-asserted-only-for-done
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
A fixer pane-process record counts as a live wait for as long as the pane process runs, so a leftover fixer strand can hold the quiet notice; its fix bounds a fixer record by the fixer's timeout, which changes the liveness rule.
No test pins that a gate or fixer registered under another run's slug leaves this run's notice alone.
The run-end stamp is asserted only in the cleaned-up done row of the shuttle wait test.
