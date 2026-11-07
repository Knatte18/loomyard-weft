---
round: 1
ledger:
  - key: openlaunch-effort-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: loadconfig-agents-yaml-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orch-reload-list-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: configrole-not-asserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: effort-not-validated-at-load
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: config-help-line-removed
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings: one MEDIUM and five LOW.
Four of them are test-coverage gaps (OpenLaunch effort, LoadConfig's agents.yaml load, the orch reload list, ConfigRole for four roles).
One asks `checkEfforts` to refuse values outside Claude Code's effort vocabulary, which changes a contract and needs fresh review.
One is a scope finding about a deleted `lym config` help line.
Only the OpenLaunch gap gates.
