---
round: 2
ledger:
  - key: gatecli-goteststargs-duplicate-test
    rounds: [1]
    status: resolved
  - key: new-tests-skip-parallel-without-global-state
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: gate-refusal-rows-untested
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: gateslot-absent-config-by-error-text
    rounds: [1]
    status: resolved
  - key: publish-failure-record-fields-unchecked
    rounds: [1]
    status: resolved
  - key: config-strictness-pattern-stale-caller-list
    rounds: [1]
    status: resolved
  - key: menu-hub-write-guard-untested
    rounds: [1]
    status: resolved
---

Round 2 found five of round 1's seven keys fixed: the duplicate `goTestArgs` test, error-text detection of an absent `gate.yaml`, publish failure record checks, the strict-caller list and the menu guard test.
Two remain.
The parallelism rule is still broken by two rewritten `cardgate_test.go` tests in `websterengine` (BLOCKING).
The gate refusal rows are covered except the absent `gate.yaml` trigger, whose CLI envelope carries two way-forward commands and has no scenario row (LOW).
