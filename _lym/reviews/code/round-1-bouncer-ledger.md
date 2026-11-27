---
round: 1
ledger:
  - key: upkeep-call-passes-running-no-marked-tests
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: config-keys-named-in-short-form
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: marked-scope-runs-tiers-whole-under-no-carry
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: non-go-hub-upkeep-call-always-refused
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cmd-lym-unmarked-repo-scans-unplaced
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: merge-danger-door-and-blast-radius-understated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: guard-moves-and-held-carry-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: added-gate-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: guard-step-drops-held-under-filter-flag
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: only-test-scan-go-list-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: dobby-doc-misplaces-upkeep-and-timeout-lines
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opens every key; nothing carries from an earlier round.

The weight sits on the new marked-scope upkeep key.
It can pass while running none of its tests, runs whole tiers under `--no-carry`, and is rendered for hubs whose scoper always refuses it.
The behaviours that hold guards off the task gates and record them in the green record have no test.
The rest is config-key naming, two unplaced scans in `cmd/lym`, the change description's merge-danger call and smaller code and doc points.
