---
round: 1
ledger:
  - key: batched-member-skips-under-lock-checks
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: member-pr-decision-outside-lock
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: landing-depends-on-lym-queue-outcome
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: backstop-loses-ledger-landings
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: leader-culprit-batched-by-next-leader
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: conflict-session-holds-parent-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: membership-not-rechecked-before-move
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: entry-lifecycle-across-halts-and-rejoins
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: detached-head-other-readers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: detached-restore-misses-earlier-rows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: checkout-leftovers-dirty-batch-gate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: records-hold-from-join-has-no-mechanism
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: acceptance-criteria-miss-added-behaviours
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: batch-gate-flaky-and-failure-records
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: bisection-blame-premises
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-counts-and-unbounded-rebuilds
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: landed-not-on-parent-bullet-wrong-cause
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: retired-key-warning-in-hub-layer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no key recurs.

The open findings cluster in four areas:

- What the leader does for a member that today's Finalize does for itself under the lock:
  owner and reconcile checks, the pull-request decision and the re-check before the move.
- Queue state and recovery: outcomes held under `.lym`, the backstop, entry lifecycle and the records hold.
- The detached leader worktree: other HEAD readers, restore on resume and checkout leftovers.
- Culprit, conflict and bisection paths, with their bounds and acceptance criteria.
