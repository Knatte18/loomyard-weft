---
round: 1
ledger:
  - key: configcli-doc-stale-retired-keys
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: merge-copy-ignores-new-webster-key
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darn-empty-model-list-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: run-settings-rename-model-keys-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four LOW findings: F1 maps to the stale configcli doc, F2 to the Merge copy, F3 to the darn empty-list test, F4 to the run_settings test.
None is gating, so the loop converged in the first round.
