---
round: 2
ledger:
  - key: batched-member-skips-under-lock-checks
    rounds: [1]
    status: resolved
  - key: member-pr-decision-outside-lock
    rounds: [1]
    status: resolved
  - key: landing-depends-on-lym-queue-outcome
    rounds: [1]
    status: resolved
  - key: backstop-loses-ledger-landings
    rounds: [1]
    status: resolved
  - key: leader-culprit-batched-by-next-leader
    rounds: [1]
    status: resolved
  - key: conflict-session-holds-parent-lock
    rounds: [1]
    status: resolved
  - key: membership-not-rechecked-before-move
    rounds: [1]
    status: resolved
  - key: entry-lifecycle-across-halts-and-rejoins
    rounds: [1]
    status: resolved
  - key: detached-head-other-readers
    rounds: [1]
    status: resolved
  - key: detached-restore-misses-earlier-rows
    rounds: [1]
    status: resolved
  - key: checkout-leftovers-dirty-batch-gate
    rounds: [1]
    status: resolved
  - key: records-hold-from-join-has-no-mechanism
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-miss-added-behaviours
    rounds: [1]
    status: resolved
  - key: batch-gate-flaky-and-failure-records
    rounds: [1]
    status: resolved
  - key: bisection-blame-premises
    rounds: [1]
    status: resolved
  - key: gate-counts-and-unbounded-rebuilds
    rounds: [1]
    status: resolved
  - key: landed-not-on-parent-bullet-wrong-cause
    rounds: [1]
    status: resolved
  - key: retired-key-warning-in-hub-layer
    rounds: [1]
    status: resolved
  - key: held-member-path-reruns-merge-and-gate
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: remaining-records-replace-own-ledger-tip
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised 18 findings, and the round 2 review marks all of them resolved.
The records-hold finding was accepted as fixed differently from its proposal:
a marker-based hold-off stands in for a pair push lock that does not exist.

Round 2 raised two BLOCKING design findings, both about what a batched run does after its work has landed:

- `held-member-path-reruns-merge-and-gate`: the held-work path chosen by the fix for the queue-outcome dependency still merges the parent and runs the full gate under the lock.
- `remaining-records-replace-own-ledger-tip`: the remaining landing records take the parent head, so the ledger tip and the dobby's landed-commit record name the wrong commit and the next run reads the parent as ahead.

The round 2 review was incremental and listed further reports below its BLOCKING bar, some of them MEDIUM,
under `## Not raised: below the incremental bar`.
They are not findings and have no ledger key.
