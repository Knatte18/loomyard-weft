---
round: 2
ledger:
  - key: parent-branch-move-not-surfaced
    rounds: [1]
    status: resolved
  - key: stopped-check-callers-unpinned
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: skip-closure-untested
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: needs-replay-named-as-stopped
    rounds: [1]
    status: resolved
  - key: landed-error-pauses-run
    rounds: [1]
    status: resolved
  - key: check-runs-after-status-commit
    rounds: [1]
    status: resolved
  - key: guard-table-rows-missing
    rounds: [1]
    status: resolved
  - key: pr-review-pushtask-pausepair-untested
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: owner-machine-background-lacks-push
    rounds: [1]
    status: resolved
  - key: owner-machine-enforcement-overstates-tests
    rounds: [1]
    status: resolved
  - key: code-repo-literal-duplicated
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: parent-notice-path-outside-shedrun
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: stopped-branches-names-own-busy-records
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: owner-machine-bullet-exceptions-narrow
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings, three of them gating.
Round 2 does not raise seven of them again and rejects the parent-branch, `landed()` and ordering concerns, so those keys are resolved.
Round 2's F3 covers the three wiring-test keys together, `stopped-check-callers-unpinned`, `skip-closure-untested` and `pr-review-pushtask-pausepair-untested`, which stay open at MEDIUM.
Round 2 adds four new keys.
F1 is `code-repo-literal-duplicated` and F2 is `parent-notice-path-outside-shedrun`, both BLOCKING under PATTERN entries.
F4 is `stopped-branches-names-own-busy-records` and F5 is `owner-machine-bullet-exceptions-narrow`, both LOW.
