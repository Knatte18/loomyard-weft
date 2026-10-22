---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-ends-finalize-past-merge
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reed-down-misses-loom-driver-strand
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d9-omits-reed-down-refusal-change
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1's sixteen findings were not re-raised in round 2, and round 2's focus pass rejected the re-checks of D5's wait, the `StartTied` ordering, remote git children, reed down's readings and D8's rule as non-defects, so all sixteen are resolved.
Round 2 found four new issues.
Two are BLOCKING design gaps in the process-end ordering: the refusal probe belongs ahead of `runproc.End` in D4's orders, and D7 must exempt a Finalize past its parent-side merge.
Two are MEDIUM consistency gaps: reed down's not-running reading misses a live loom driver strand, and D9 does not record that `reed down` stops reaching the session end for a running run.
