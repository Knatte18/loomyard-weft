---
round: 1
ledger:
  - key: reset-refusal-local-tags-and-pending-teardowns
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: existing-hub-advice-partial-clone
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reset-refusal-misses-prime-uncommitted-stash-detached
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: doc-overstates-settle-callers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: sweep-failed-tag-push-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: catchup-no-pair-call-site-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: summary-merge-danger-understated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: remove-settle-remote-guard-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: board-push-auth-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: teardown-child-duplicate-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: settled-removal-reports-no-mutations
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: settle-drops-stray-path
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sweep-landed-check-trusts-other-local-branches
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: remove-cli-settle-wiring-undriven
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sandbox-reset-scenario-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-reset-network-order-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: move-aside-way-forward-breaks-worktrees
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 opened every key above; there is no earlier ledger.
The findings cluster in the clone's `--reset` refusal and advice, the gone-pair teardown settle,
the cleanup sweep's remote branch deletion, test coverage and the change description.
