---
round: 1
ledger:
  - key: new-error-paths-no-way-forward-no-spec-row
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: transient-refusals-without-spec-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: migrateconfig-orientation-insert-index
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: run-help-dash-reads-as-flag
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised four findings (F1 to F4), all new.
F1 is the gate: the reset's after-archive uncommitted-path read and `overviewFrameHash` errors name no way forward and have no spec row.
F2 is the sibling gap for the Merriam base, recover-batch and rebaseline reads.
F3 is the `MigrateConfig` insert index when both retired keys are present.
F4 is the dash in the `run` long help.
