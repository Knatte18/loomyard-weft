---
round: 2
ledger:
  - key: pairless-run-reconcile-errors
    rounds: [1]
    status: resolved
  - key: remove-snapshot-not-taken-in-origin-mode
    rounds: [1]
    status: resolved
  - key: origin-task-branch-survives-cancel
    rounds: [1]
    status: resolved
  - key: owned-process-end-outside-cancel-phase
    rounds: [1]
    status: resolved
  - key: reconcile-moves-live-pair
    rounds: [1]
    status: resolved
  - key: ended-verdict-undecided
    rounds: [1]
    status: resolved
  - key: unreachable-origin-unbounded
    rounds: [1]
    status: resolved
  - key: start-can-seed-during-cancel
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: interrupted-remove-cannot-rerun
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: windows-end-detects-nothing
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: status-lock-across-network-gate-hibernate
    rounds: [1]
    status: resolved
  - key: cleanupslug-lock-conflict
    rounds: [1]
    status: resolved
  - key: two-run-locks-one-name
    rounds: [1]
    status: resolved
  - key: d7-misplaces-board-push
    rounds: [1]
    status: resolved
  - key: wrong-tackler-marker-path
    rounds: [1]
    status: resolved
  - key: tackler-marker-read-before-end
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: present-absent-split-differs-d2-d4
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: windows-pid-only-liveness
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: kept-pair-rerun-goes-to-tackler
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: done-kept-pair-unremovable
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: leftover-task-branch-reads-nothing-to-cancel
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: marker-check-after-owner-check
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: end-only-step9-actions-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: held-lock-not-always-raced-batten
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: one-archive-tag-two-tips
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: remote-not-named-on-removal
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's 15 keys were all open.
Twelve do not recur in round 2 and are resolved.
Three recur as round-2 findings: `start-can-seed-during-cancel` as BLOCKING F1 (the unregistered step or `--window` batten), `interrupted-remove-cannot-rerun` as F3 (the `--remove` re-run stops at step 6 on `ErrPairNotFound`), and `windows-end-detects-nothing` as F6 (the Windows list runs before the marker).
Eleven keys are new in round 2, four of them LOW.
