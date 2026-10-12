---
round: 2
ledger:
  - key: d10-handoff-keeps-newest-not-lasthandoff
    rounds: [1]
    status: resolved
  - key: d9-scratch-deletes-lock-files-and-races-batten-run
    rounds: [1]
    status: resolved
  - key: d9-tracked-but-deleted-no-done-check
    rounds: [1]
    status: resolved
  - key: d1-mode-0700-fails-on-windows
    rounds: [1]
    status: resolved
  - key: d4-windows-redirect-and-removal-failure-undecided
    rounds: [1]
    status: resolved
  - key: goal-no-testmain-packages-leak-on-kill
    rounds: [1]
    status: resolved
  - key: d6-allowlist-unbounded-and-disagrees-with-q3
    rounds: [1]
    status: resolved
  - key: d7-in-process-segments-skip-verb-refusals
    rounds: [1]
    status: resolved
  - key: d11-board-read-failure-unbounded
    rounds: [1]
    status: resolved
  - key: d11-scratch-sole-declarer-claim-false
    rounds: [1]
    status: resolved
  - key: d2-sweep-check-order-open
    rounds: [1]
    status: resolved
  - key: d3-adopted-root-no-trust-check
    rounds: [1]
    status: resolved
  - key: d9-orphan-rule-fails-open-on-stat-error
    rounds: [1]
    status: resolved
  - key: d10-cleanup-not-atomic-against-watcher
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-orphan-rule-overlaps-tracked-but-deleted
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d9-status-lock-span-misses-resume-branch
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: d4-tmux-isolation-allowlist-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: support-log-q5-interview6-disagree-with-d9-d10
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised 13 findings, two BLOCKING.
Round 2 re-raises none of them; the review checks the Windows, allowlist, board-read, lock-file and `LastHandoff` points and finds them settled.
Round 2 raises 5 new findings, none BLOCKING.
The gating one is F1 (`d10-cleanup-not-atomic-against-watcher`, design, MEDIUM): hold the orch state write lock from the phase read through the last removal.
Its key is first raised in round 2, so it is carried to Open risks rather than blocking.
The rest are D9's orphan/tracked-but-deleted overlap, the status-lock span in the resume branch, D4's scan allowlist bound, and stale support-log text in Q5 and Interview 6.
