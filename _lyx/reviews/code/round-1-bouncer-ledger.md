---
round: 1
ledger:
  - key: usage-txt-missing-window-percentages
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: config-toml-byte-identical-unevidenced
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d5-grading-unchecked-rows
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: grid-no-70-percent-stop
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: blocking-counts-not-recomputable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: workspace-write-reads-unconfined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: grid-skips-refused-cells
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: conclusion-code-checked-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fill-ampersand-corruption
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: grid-stale-rollout-readout
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: committed-scripts-not-runnable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: report-semantic-line-breaks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised twelve findings, none from an earlier round.
Five design-class MEDIUM findings gate: missing window percentages, unevidenced config.toml hash, unrecomputable BLOCKING counts, unconfined reads, and refused cells skipped on rerun.
