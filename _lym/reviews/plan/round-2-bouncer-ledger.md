---
round: 2
ledger:
  - key: fabric-push-bypasses-records-hold
    rounds: [1]
    status: resolved
  - key: beam-parent-push-ceiling-unpassable
    rounds: [1]
    status: resolved
  - key: ledger-removals-drop-landings
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: held-only-pair-never-level
    rounds: [1]
    status: resolved
  - key: unrecorded-landing-refusal-dead-way-forward
    rounds: [1]
    status: resolved
  - key: landing-state-only-in-lym-cache
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: beam-pushes-unrecorded-landing
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pushes-off-landing-blocks-removal
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: dropped-landing-forgotten-while-commit-stands
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: spawns-skip-push-interval
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: board-spawn-line-refused-by-verb
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: summary-misses-beam-waits
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: catch-up-test-waits-production-duration
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 2 has fewer findings than round 1, all BLOCKING.
Most round 1 keys are resolved.

Two stay open:

- `ledger-removals-drop-landings`: ordinary writers of a pair's entry set neither `Slug` nor `Worktree`, so a drop's held commit cannot be found.
- `landing-state-only-in-lym-cache`: the rebuild from git still releases records a lost ledger held.

The new keys cluster on the records hold and the landing's state (removal, beam pusher, pushes off, dropped landing),
with the rest on pusher spawns, the push summary's coverage of waits and one slow test.

Outside the ledger, the round 2 review rejected two items as off the changed hunks that its reviewer wants the parent to read:
batten's driver is not wired with the network wait ceiling or wait record path,
and card 10's refusal depends on card 18 to pass landing commits through.
