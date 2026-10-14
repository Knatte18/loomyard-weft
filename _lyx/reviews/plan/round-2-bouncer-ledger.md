---
round: 2
ledger:
  - key: null-per-perch-budget-breaks-config-set
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: eligible-stop-hold-drops-on-arbiter-circling-decision
    rounds: [1]
    status: resolved
  - key: reviewbudget-loaded-config-test-at-zero
    rounds: [1]
    status: resolved
  - key: zero-to-null-migration-skips-sparse-worktree-files
    rounds: [1]
    status: resolved
  - key: routing-projection-reports-engine-default-for-zero-round
    rounds: [1]
    status: resolved
  - key: darn-unparseable-document-refusal-unreachable
    rounds: [1]
    status: resolved
  - key: yamlengine-set-change-bundled-into-loom-config-card
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: checklocal-passes-null-for-run-wide-budget-key
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hold-reading-test-circling-decision-layout-unnamed
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, two BLOCKING.
Round 2 no longer raises the eligible-stop hold, the loaded-config test, the migration scope, the routing projection or the darn refusal; all five are resolved.
The null-budget `--set` finding reappears narrowed to the reverse direction (setting back to null over an integer leaf), now MEDIUM scope.
Round 2 adds three LOW findings: the yamlengine change bundled into card 3, `CheckLocal` passing null for run-wide keys, and the unnamed circling-decision file layout in card 7's hold-window test row.
