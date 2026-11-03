---
round: 1
ledger:
  - key: parent-branch-move-not-surfaced
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stopped-check-callers-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: skip-closure-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: needs-replay-named-as-stopped
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: landed-error-pauses-run
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: check-runs-after-status-commit
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
  - key: guard-table-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pr-review-pushtask-pausepair-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: owner-machine-background-lacks-push
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: owner-machine-enforcement-overstates-tests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings: F1 `parent-branch-move-not-surfaced`, F2 `stopped-check-callers-unpinned`, F3 `skip-closure-untested`, F4 `needs-replay-named-as-stopped`, F5 `landed-error-pauses-run`, F6 `check-runs-after-status-commit`, F7 `guard-table-rows-missing`, F8 `pr-review-pushtask-pausepair-untested`, F9 `owner-machine-background-lacks-push`, F10 `owner-machine-enforcement-overstates-tests`.
F1, F2 and F3 gate.
F10 depends on F2, F3 and F8 landing their tests.
