---
round: 2
ledger:
  - key: ended-phases-literal-copy
    rounds: [1]
    status: resolved
  - key: hibernate-daemon-live-untested
    rounds: [1]
    status: resolved
  - key: hibernate-dir-missing-at-watch-open
    rounds: [1]
    status: resolved
  - key: resuming-takeover-skips-reconcile-notice
    rounds: [1]
    status: resolved
  - key: cancelled-sequence-keeps-carrier-busy
    rounds: [1]
    status: resolved
  - key: stale-daemon-comment-cleanup-test
    rounds: [1]
    status: resolved
  - key: reexec-hold-ends-before-notice
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised six findings; round 2 rejected every round 1 focus as not a defect, so all six are resolved.
Round 2 raised one new finding, F1, mapped to `reexec-hold-ends-before-notice`.
Its fix text names the exact change, so it is mechanical.
