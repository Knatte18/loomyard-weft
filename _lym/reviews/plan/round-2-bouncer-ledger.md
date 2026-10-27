---
round: 2
ledger:
  - key: adopt-seed-reaches-every-migrated-document
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: no-test-reaches-top-of-loop-worktree-gone-exit
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: after-tick-read-race-branch-unreached
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: trace-test-row-breaks-hub-for-later-rows
    rounds: [1]
    status: resolved
  - key: card7-extends-doc-it-rewrites
    rounds: [1]
    status: resolved
  - key: beam-status-readcheck-writes-scratch-dir
    rounds: [1]
    status: resolved
  - key: refused-tier-leaves-running-set
    rounds: [1]
    status: resolved
  - key: post-tick-record-read-recreates-stub
    rounds: [1]
    status: resolved
  - key: fatal-stop-line-no-way-forward
    rounds: [1]
    status: resolved
  - key: trace-test-rows-recover-existing-test-inputs
    rounds: [1]
    status: resolved
  - key: gate-keys-table-and-adopt-scratch-doc-stale
    rounds: [1]
    status: resolved
  - key: empty-name-guards-unreachable
    rounds: [1]
    status: resolved
  - key: busy-slots-cannot-end-waiting-gate
    rounds: [1]
    status: resolved
---

Round 1 raised twelve keys; round 2 raised three findings.
The top-of-iteration worktree-gone exit in card 8 is still unpinned (F1), and the round 1 fix to the after-tick read left a race branch no row reaches plus a contradictory error clause (F2, new key).
The adopt seed shrank to a LOW remainder: a hub `gate.yaml` seeded from a prime copy or legacy files gets no `adopt` until a later build (F3).
The other round 1 keys were not raised again and are resolved.
`post-tick-record-read-recreates-stub` is resolved: the round 2 review checked that the reads recreate no directory.
