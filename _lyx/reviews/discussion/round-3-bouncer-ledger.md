---
round: 3
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
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: interrupted-remove-cannot-rerun
    rounds: [1, 2]
    status: resolved
  - key: windows-end-detects-nothing
    rounds: [1, 2, 3]
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
    status: resolved
  - key: present-absent-split-differs-d2-d4
    rounds: [2]
    status: resolved
  - key: windows-pid-only-liveness
    rounds: [2]
    status: resolved
  - key: kept-pair-rerun-goes-to-tackler
    rounds: [2]
    status: resolved
  - key: done-kept-pair-unremovable
    rounds: [2]
    status: resolved
  - key: leftover-task-branch-reads-nothing-to-cancel
    rounds: [2]
    status: resolved
  - key: marker-check-after-owner-check
    rounds: [2]
    status: resolved
  - key: end-only-step9-actions-unstated
    rounds: [2]
    status: resolved
  - key: held-lock-not-always-raced-batten
    rounds: [2]
    status: resolved
  - key: one-archive-tag-two-tips
    rounds: [2]
    status: resolved
  - key: remote-not-named-on-removal
    rounds: [2]
    status: resolved
  - key: tackler-started-unrecorded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: weft-leftovers-not-refused-at-start
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: nothing-left-refusal-misses-remove
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 3 has five findings.
Two sit on keys open since round 1: `start-can-seed-during-cancel` (F1, BLOCKING: the second gate is behind batten's pre-run hooks, which seed status without the run lock) and `windows-end-detects-nothing` (F3: Windows `End` returns nothing, so the survivor refusal has no producer).
Three are new: `tackler-started-unrecorded` (F2), `weft-leftovers-not-refused-at-start` (F4) and `nothing-left-refusal-misses-remove` (F5, LOW).
Every other round 2 key does not recur and is resolved; the review's Rejected section confirms the `--remove` re-run and the marker-read ordering.
