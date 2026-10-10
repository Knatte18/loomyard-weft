---
round: 1
ledger:
  - key: gatecli-goteststargs-duplicate-test
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: new-tests-skip-parallel-without-global-state
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: gate-refusal-rows-untested
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: gateslot-absent-config-by-error-text
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: publish-failure-record-fields-unchecked
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: config-strictness-pattern-stale-caller-list
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: menu-hub-write-guard-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised seven findings, all open.
Three are BLOCKING: a per-helper test duplicating the CLI scenario (F1), new tests skipping `t.Parallel` without naming global state (F2), and gate refusal rows no test reaches (F3).
Four are LOW: absent `gate.yaml` detected by error text (F4), publish failure record fields reaching the prompt and command line unchecked (F5), a stale strict-caller list (F6), and an untested menu guard wiring (F7).
