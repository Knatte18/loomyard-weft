---
round: 3
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
    status: resolved
  - key: remaining-records-replace-own-ledger-tip
    rounds: [2]
    status: resolved
---

Round 1 raised 18 findings, all resolved by round 2.
The records-hold finding was accepted as fixed differently from its proposal:
a marker-based hold-off stands in for a pair push lock that does not exist.

Round 2 raised two BLOCKING design findings about what a batched run does after its work has landed.
Round 3 marks both resolved and raises no finding:

- `held-member-path-reruns-merge-and-gate`: a run whose work the parent holds takes a held path with no parent merge and no gate.
- `remaining-records-replace-own-ledger-tip`: the remaining records get the run's own landing commit.
  This was fixed differently from the review's proposal.

The round 2 and round 3 reviews were incremental and listed further reports below their BLOCKING bar,
some of them MEDIUM, under `## Not raised: below the incremental bar`.
They are not findings and have no ledger key.
