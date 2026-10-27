---
round: 3
ledger:
  - key: after-tick-read-error-shares-tick-failure-episode
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: after-tick-read-no-record-branch-unreached
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: adopt-seed-reaches-every-migrated-document
    rounds: [1, 2]
    status: resolved
  - key: no-test-reaches-top-of-loop-worktree-gone-exit
    rounds: [1, 2]
    status: resolved
  - key: after-tick-read-race-branch-unreached
    rounds: [2]
    status: resolved
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

Round 1 raised twelve findings, round 2 three, round 3 two.
Every earlier key was not raised again in round 3 and is resolved.
The two round 3 findings are LOW and local to card 8's after-tick-read paragraph: the read error should use an episode of its own, and the no-record branch needs a test row.
Both carry a fix the review text already fixes, so both are `mechanical`.
