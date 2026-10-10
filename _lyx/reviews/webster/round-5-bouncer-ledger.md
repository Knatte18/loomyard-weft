---
round: 5
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
    status: resolved
  - key: unloadable-gate-yaml-no-way-forward
    rounds: [4]
    status: resolved
  - key: gate-way-forward-untested-at-sites
    rounds: [5]
    status: open
    class: scope
    severity: MEDIUM
  - key: plan-gate-acquire-failure-degrades-silently
    rounds: [5]
    status: open
    class: decision
    severity: LOW
  - key: validate-verbs-wired-index-unpinned
    rounds: [5]
    status: open
    class: scope
    severity: LOW
  - key: slotted-plan-gate-row-redundant
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 to 3 closed every earlier key.
The two round-4 keys are resolved: the production fixes landed and round 5 confirms every path reaches a slotted index and every acquisition error carries its way forward.
Round 5 raised four new non-gating keys.
F1 maps to `gate-way-forward-untested-at-sites`: no test reaches the `gate.yaml` way forward through the `hubgeom`, card-verify and `verifytree.Verify` sites.
F2 maps to `plan-gate-acquire-failure-degrades-silently`: the plan gate's type load degrades to the scan on an acquire failure, with no stated decision.
F3 maps to `validate-verbs-wired-index-unpinned`: no test pins the webster and loom validate verbs to the wired index.
F4 maps to `slotted-plan-gate-row-redundant`: the `plan gate pass` slotted row repeats the `Validate` row.
