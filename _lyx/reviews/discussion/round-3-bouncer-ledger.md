---
round: 3
ledger:
  - key: bolt-replay-layout-premise
    rounds: [1]
    status: resolved
  - key: ended-means-removed-from-origin
    rounds: [1, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: wake-paused-run-not-moved
    rounds: [1]
    status: resolved
  - key: ended-set-outside-reconcile
    rounds: [1]
    status: resolved
  - key: cleanup-dry-run-mutates
    rounds: [1, 2]
    status: resolved
  - key: prime-lock-wait-unbounded
    rounds: [1]
    status: resolved
  - key: fabricengine-seams-missing
    rounds: [1]
    status: resolved
  - key: notice-lost-without-orch
    rounds: [1]
    status: resolved
  - key: set-aside-tags-collide-with-archive
    rounds: [1]
    status: resolved
  - key: gone-class-source-stale
    rounds: [1, 2]
    status: resolved
  - key: push-deferral-loops-or-silent
    rounds: [1]
    status: resolved
  - key: own-run-exclusion-resume-unreachable
    rounds: [1]
    status: resolved
  - key: ended-busy-gets-resume-way-forward
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: code-branch-rewrite-misread
    rounds: [1]
    status: resolved
  - key: destruction-chokepoint-covers-tags
    rounds: [1]
    status: resolved
  - key: finalize-parent-lock-order
    rounds: [1]
    status: resolved
  - key: d9-bound-ignores-set-aside
    rounds: [1]
    status: resolved
  - key: notice-signer-not-always-fabric
    rounds: [1]
    status: resolved
  - key: bolt-two-push-locks-unnamed
    rounds: [1]
    status: resolved
  - key: gone-ended-weft-false-alarm
    rounds: [1]
    status: resolved
  - key: q8-contradicts-d8
    rounds: [1]
    status: resolved
  - key: goal-cites-wrong-section
    rounds: [1]
    status: resolved
  - key: wake-reconcile-held-by-own-hub-pause
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: records-lock-waits-unbounded
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bolt-replay-meets-board-validation
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: bolt-empty-commit-readme-stamp
    rounds: [2]
    status: resolved
  - key: pull-first-missing-from-ended-set-callers
    rounds: [2]
    status: resolved
  - key: rewound-rewritten-overlap
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: gone-push-mails-before-done-mark
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 3 raised seven findings: two BLOCKING (one design, one decision), four MEDIUM, one LOW.
Key-to-finding map: F1 `bolt-replay-meets-board-validation`, F2 `ended-busy-gets-resume-way-forward`, F3 `wake-reconcile-held-by-own-hub-pause`, F4 `ended-means-removed-from-origin`, F5 `rewound-rewritten-overlap`, F6 `records-lock-waits-unbounded`, F7 `gone-push-mails-before-done-mark`.

Four round 2 keys carried over and reopened: the Bolt replay against the board's write validation (now the lock the reconcile already holds and the unit of refusal), the busy ended pair (now a dirty worktree blocks the teardown), the wake under a hub pause (now a pause the request does not own), and the records-branch lock waits (now the teardown's own waits).
`ended-means-removed-from-origin` reopened from round 1 on its anchor evidence.
The dry-run, gone-class source, README stamp and pull-first keys were not raised again and are resolved.
Findings fell from 22 to 8 to 7, but the Bolt replay and busy-pair keys still carry BLOCKING findings.
