---
round: 3
ledger:
  - key: gatecli-goteststargs-duplicate-test
    rounds: [1]
    status: resolved
  - key: new-tests-skip-parallel-without-global-state
    rounds: [1, 2]
    status: resolved
  - key: gate-refusal-rows-untested
    rounds: [1, 2]
    status: resolved
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
  - key: test-files-not-gofmt-clean
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: reconcile-help-omits-gate-yaml
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 3 closed the last two round-2 keys: the `cardgate_test.go` tests all run in parallel, and the absent `gate.yaml` refusal names `lyx fabric reconcile` once, with a scenario row pinning it.
Two new LOW consistency findings remain: F1 maps to `test-files-not-gofmt-clean` and F2 to `reconcile-help-omits-gate-yaml`.
