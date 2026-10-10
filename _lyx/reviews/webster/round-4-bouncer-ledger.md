---
round: 4
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
    status: resolved
  - key: reconcile-help-omits-gate-yaml
    rounds: [3]
    status: resolved
  - key: whole-plan-validate-unslotted-type-load
    rounds: [4]
    status: open
    class: scope
    severity: BLOCKING
  - key: unloadable-gate-yaml-no-way-forward
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
---

Rounds 1 to 3 closed every earlier key; the two round-3 LOW keys did not return in round 4.
Round 4 raised two new keys.
F1 maps to `whole-plan-validate-unslotted-type-load`: `planglyph.Validate` loads types through the unslotted default loader, reached from `lyx webster validate` and `lyx loom validate-plan --require-approved`.
F2 maps to `unloadable-gate-yaml-no-way-forward`: a `gate.yaml` that parses with an invalid value halts the Go-side gate sites with no way forward and no refusal-spec row.
