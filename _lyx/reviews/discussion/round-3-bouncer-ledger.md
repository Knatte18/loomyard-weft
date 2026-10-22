---
round: 3
ledger:
  - key: d9-duplicates-reed-reap
    rounds: [1]
    status: resolved
  - key: d5-gate-wait-unreadable
    rounds: [1]
    status: resolved
  - key: end-before-loop-kills-merge-step
    rounds: [1]
    status: resolved
  - key: remote-git-children-escape-end
    rounds: [1]
    status: resolved
  - key: reed-down-readings-undefined
    rounds: [1]
    status: resolved
  - key: d7-misses-batten-owner-probe
    rounds: [1]
    status: resolved
  - key: stale-running-batten-record
    rounds: [1]
    status: resolved
  - key: d8-deleted-cwd-reaps-live-pair
    rounds: [1]
    status: resolved
  - key: driver-lock-recheck-callers
    rounds: [1]
    status: resolved
  - key: pattern-text-vs-d5-skip-readings
    rounds: [1]
    status: resolved
  - key: d10-sixty-minute-evidence
    rounds: [1]
    status: resolved
  - key: reap-signal-unstated
    rounds: [1]
    status: resolved
  - key: d11-open-risk-shell-end
    rounds: [1]
    status: resolved
  - key: windows-remove-paused-refuses
    rounds: [1]
    status: resolved
  - key: support-log-stale-d-numbers
    rounds: [1]
    status: resolved
  - key: lift-fact-names-wrong-function
    rounds: [1]
    status: resolved
  - key: d4-end-before-refusal-probe
    rounds: [2]
    status: resolved
  - key: d7-ends-finalize-past-merge
    rounds: [2]
    status: resolved
  - key: reed-down-misses-loom-driver-strand
    rounds: [2]
    status: resolved
  - key: d9-omits-reed-down-refusal-change
    rounds: [2]
    status: resolved
  - key: d7-revival-does-not-restart-driver
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-push-child-coverage-reading
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d4-gone-worktree-order
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d4-bound-overstates-d5
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-end-no-bound-after-sigkill
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1's sixteen findings were resolved by round 2, and round 2's four findings were resolved by round 3: its focus pass rejected the re-checks of the refusal-probe order, the Finalize exception's three sites, the loom driver strand reading and the D9 refusal change.
Round 3 found five new issues, none on an earlier key.
The BLOCKING one is D7's revival sentence, which names a batten mechanism that revives only halted or awaiting children with a dead strand and so does not restart a Go driver chain that `End` ended.
The two MEDIUM ones are D6's missing coverage reading for a marked push child under D7's Finalize exception, and D4's missing order for a pair whose task worktree is gone.
The LOW ones are D4's Bound overstating what D5 allows and D3's missing bound and outcome after SIGKILL.
