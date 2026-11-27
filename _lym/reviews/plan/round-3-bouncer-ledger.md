---
round: 3
ledger:
  - key: fabric-push-bypasses-records-hold
    rounds: [1]
    status: resolved
  - key: beam-parent-push-ceiling-unpassable
    rounds: [1]
    status: resolved
  - key: ledger-removals-drop-landings
    rounds: [1, 2]
    status: resolved
  - key: held-only-pair-never-level
    rounds: [1]
    status: resolved
  - key: unrecorded-landing-refusal-dead-way-forward
    rounds: [1]
    status: resolved
  - key: landing-state-only-in-lym-cache
    rounds: [1, 2]
    status: resolved
  - key: tests-stand-in-for-repo-modules
    rounds: [1]
    status: resolved
  - key: pushledger-tests-lack-tmuxkit-main
    rounds: [1]
    status: resolved
  - key: archive-parent-merge-left-test-only
    rounds: [1]
    status: resolved
  - key: beam-pr-head-missing-on-origin
    rounds: [1]
    status: resolved
  - key: spawn-owner-test-row-breaks
    rounds: [1]
    status: resolved
  - key: dobbyengine-doc-stale
    rounds: [1]
    status: resolved
  - key: pairteardown-doc-stale
    rounds: [1]
    status: resolved
  - key: landing-records-commit-before-hold
    rounds: [1]
    status: resolved
  - key: spawn-criterion-tests-incomplete
    rounds: [1]
    status: resolved
  - key: pattern-shed-driver-background-stale
    rounds: [1]
    status: resolved
  - key: cancel-intent-committed-before-landing-refusal
    rounds: [1]
    status: resolved
  - key: overview-applies-to-misplaced
    rounds: [1]
    status: resolved
  - key: hub-clear-time-unset-before-first-clear
    rounds: [1]
    status: resolved
  - key: removal-pushes-held-records
    rounds: [2]
    status: resolved
  - key: beam-pushes-unrecorded-landing
    rounds: [2]
    status: resolved
  - key: pushes-off-landing-blocks-removal
    rounds: [2]
    status: resolved
  - key: dropped-landing-forgotten-while-commit-stands
    rounds: [2]
    status: resolved
  - key: spawns-skip-push-interval
    rounds: [2]
    status: resolved
  - key: board-spawn-line-refused-by-verb
    rounds: [2]
    status: resolved
  - key: summary-misses-beam-waits
    rounds: [2]
    status: resolved
  - key: catch-up-test-waits-production-duration
    rounds: [2]
    status: resolved
  - key: board-pusher-never-commits-writes
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: non-squash-landing-has-no-trailer-commit
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: done-write-judged-on-unpulled-bolt
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: dropped-landing-refusal-never-cleared
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: batten-driver-network-waits-invisible
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: catch-up-test-sleeps-push-backoff
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Findings fall each round, and each round's earlier findings have been reported resolved by the next.
All open keys are first raised in round 3, all BLOCKING design findings.

They cluster on three areas:

- The landing's git trace and the parent pusher: a non-squash landing leaves no trailer commit,
  the `done` write reads an unpulled Bolt, and a dropped landing's refusal is never cleared.
- Pusher and wait coverage: board writes are never committed, and batten's own driver waits are unwired and unread.
- Test speed: the catch-up test still sleeps through the push retry backoff, after its lock wait was shortened.

Outside the ledger, the round 3 review lists items it did not raise because they are off the changed hunks or below BLOCKING.
Two it marks as serious enough for the parent to read:
card 14's single shared scratch repository for ignore evaluation can be overwritten by concurrent runs,
and card 12's `TestLandingDeps_LocalLanding` expects done with origin unreachable, where the parent fetch would stop first.
Both are in the round 4 focus.
