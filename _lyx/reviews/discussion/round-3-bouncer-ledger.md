---
round: 3
ledger:
  - key: d7-runproc-end-kills-passes
    rounds: [1, 2]
    status: resolved
  - key: prime-anchor-change-strands-other-machines
    rounds: [1]
    status: resolved
  - key: d7-recovery-states-indistinguishable
    rounds: [1, 2]
    status: resolved
  - key: reconcile-premise-false-in-recovery
    rounds: [1]
    status: resolved
  - key: uncommitted-move-undo-unspecified
    rounds: [1]
    status: resolved
  - key: ended-dobbies-not-restarted-after-kill
    rounds: [1]
    status: resolved
  - key: automatic-dobby-starts-break-lock-bound
    rounds: [1, 2]
    status: resolved
  - key: child-run-landings-during-move
    rounds: [1, 2]
    status: resolved
  - key: pull-refusal-incoming-tip-undefined
    rounds: [1]
    status: resolved
  - key: reconcile-marker-commits-never-pushed
    rounds: [1]
    status: resolved
  - key: census-blind-to-unfetched-branches
    rounds: [1]
    status: resolved
  - key: empty-marker-census-unstated
    rounds: [1]
    status: resolved
  - key: stale-fabric-anchor-breaks-migrated-hub
    rounds: [1]
    status: resolved
  - key: fresh-bind-probe-cannot-see-main-weft
    rounds: [1]
    status: resolved
  - key: empty-marker-refusal-after-step-1
    rounds: [1]
    status: resolved
  - key: d6-rerun-skip-underspecified
    rounds: [1]
    status: resolved
  - key: not-landed-recovery-deletes-lyx
    rounds: [2]
    status: resolved
  - key: add-from-moving-worktree-wrong-anchor
    rounds: [2]
    status: resolved
  - key: creel-daemon-refusal-no-way-forward
    rounds: [2]
    status: resolved
  - key: markerless-tip-compared-as-different-marker
    rounds: [2]
    status: resolved
  - key: checkout-way-forward-reanchors-left-branch
    rounds: [2]
    status: resolved
  - key: conflict-way-forward-drops-later-commits
    rounds: [2]
    status: resolved
  - key: dirtiness-claim-no-first-run-check
    rounds: [2]
    status: resolved
  - key: markerless-anchor-disagrees-with-d5
    rounds: [2]
    status: resolved
  - key: acceptance-overclaims-spared
    rounds: [2]
    status: resolved
  - key: origin-reconcile-moves-weft-past-d10
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: checkout-refusal-set-target-untracked
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: replayed-commit-parent-marker-markerless
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: move-lock-refusal-pid-unrecorded
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-omits-live-dobby-pass-gate
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised sixteen and thirteen findings in D5, D7, D8 and D10.
Round 3 raised five, and none maps to an earlier key, so every key from rounds 1 and 2 is resolved.

The one BLOCKING finding is `origin-reconcile-moves-weft-past-d10`.
The origin reconcile's moving half (`reconcileRecordsBranch` through `ffWorktree` and `replayWeft`, run hub-wide from `lyx orch start`, `lyx adopt`, `lyx hibernate` and `lyx cleanup`) moves a checked-out weft with no marker comparison and no check for a standing move record.
D10 stops the pull and the replaying push only.
The fix adds a third stopping path.

`checkout-refusal-set-target-untracked` is a LOW design finding: the checkout refusal's `--set` way forward can itself be refused.
The three consistency keys are local text corrections: a markerless parent tree of a replayed commit, a move-lock refusal naming a pid no record holds, and an acceptance criterion missing the live dobby's pass gate.
