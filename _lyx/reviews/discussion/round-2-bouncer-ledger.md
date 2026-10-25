---
round: 2
ledger:
  - key: d7-runproc-end-kills-passes
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: prime-anchor-change-strands-other-machines
    rounds: [1]
    status: resolved
  - key: d7-recovery-states-indistinguishable
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: child-run-landings-during-move
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: add-from-moving-worktree-wrong-anchor
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: creel-daemon-refusal-no-way-forward
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: markerless-tip-compared-as-different-marker
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: checkout-way-forward-reanchors-left-branch
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: conflict-way-forward-drops-later-commits
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dirtiness-claim-no-first-run-check
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: markerless-anchor-disagrees-with-d5
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-overclaims-spared
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised sixteen findings, all in D7, D8, D10, D4 and D6.
Round 2 re-raised four of those keys, and I marked the other twelve resolved because no round-2 finding maps to them.

The reopened keys are the core of D7: the parent lock is sampled and not held (`child-run-landings-during-move`, BLOCKING), step 2 kills passes the wait never saw (`d7-runproc-end-kills-passes`, BLOCKING), adopt's own starters run unbounded (`automatic-dobby-starts-break-lock-bound`), and the integration landed test reads a stale tip (`d7-recovery-states-indistinguishable`).
Round 2 adds nine keys first raised there.
One is BLOCKING: the Not-landed recovery deletes the worktree's `.lyx` (`not-landed-recovery-deletes-lyx`).
The others cover D5's `Add`, D7's prime refusals, D10's markerless comparison, checkout and conflict way forwards, and three consistency items.
