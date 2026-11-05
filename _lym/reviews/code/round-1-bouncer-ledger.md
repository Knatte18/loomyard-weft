---
round: 1
ledger:
  - key: deviations-md-stale-renamed-file-name
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: commands-md-step0-cp-old-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: docslink-test-topic-level-data-row-missing
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised three low-weight findings and no gating one.
F1 maps to `deviations-md-stale-renamed-file-name`: `deviations.md` still cites the renamed study file.
F2 maps to `commands-md-step0-cp-old-path`: the step-0 `cp` in `COMMANDS.md` keeps the pre-move path while line 101 was rewritten.
F3 maps to `docslink-test-topic-level-data-row-missing`: no test row pins a `data/` folder directly under a topic folder.
