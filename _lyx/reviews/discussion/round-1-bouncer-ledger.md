---
round: 1
ledger:
  - key: command-keys-settable-from-task-branch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: remove-lock-inside-deleted-tree
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: local-remove-guard-makes-force-routine
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: late-child-recreates-removed-worktree-dir
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: absorbing-lock-premise-false
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: remove-push-lock-wait-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: noop-transition-skip-not-existing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: detached-child-loses-pause-slug
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: force-contradicts-warp-branch-gate
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: guard-order-vs-pending-records-commit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cleanup-publishes-local-orphan-weft
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: batten-env-setting-unwired
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: under-100ms-criterion-vs-d2-cut
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: window-override-no-mechanism
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: cmd-percent-expansion-in-quotes
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: half-marked-pair-remove-undefined
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: skip-in-detached-child-unreportable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-risks-carry-no-risk
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 found 18 findings, none carried from an earlier round.
The three BLOCKING ones are the command keys' trust class (F1), the push lock living inside the tree `Remove` deletes (F2), and the local remove guard making `--force` routine (F3).
The MEDIUM findings cluster on D1's premises about current push behaviour (F4 to F8), D3's remove and cleanup semantics (F9 to F11) and D4's batten wiring (F12), plus one criterion/decision mismatch (F13).
All keys are open pending round 2.
