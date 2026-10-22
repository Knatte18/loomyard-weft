---
round: 4
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
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-push-child-coverage-reading
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-gone-worktree-order
    rounds: [3]
    status: resolved
  - key: d4-bound-overstates-d5
    rounds: [3]
    status: resolved
  - key: d3-end-no-bound-after-sigkill
    rounds: [3]
    status: resolved
---

Rounds 1 and 2 are closed.
Round 3's gone-worktree order, D4 bound and SIGKILL findings did not return, and round 4 rejected the first and last explicitly.
Two round-3 keys are still open, both narrower than before.
`d7-revival-does-not-restart-driver` fell from BLOCKING to MEDIUM: round 4's F1 finds that `lyx loom resume` refuses a running child with no driver, and F2 finds that only the owner-probe site's restart is pinned, not the wake and `lyx orch start` sites, where `lyx adopt`'s `ResumeInWindow` is the mechanism.
`d6-push-child-coverage-reading` moved from decision to design: F3 finds the parent lock record is read live and released at Finalize Done, so a push can outlive its exemption.
