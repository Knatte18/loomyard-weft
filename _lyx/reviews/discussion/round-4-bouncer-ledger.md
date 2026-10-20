---
round: 4
ledger:
  - key: bolt-replay-layout-premise
    rounds: [1]
    status: resolved
  - key: ended-means-removed-from-origin
    rounds: [1, 3]
    status: resolved
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
    status: resolved
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
    status: resolved
  - key: records-lock-waits-unbounded
    rounds: [2, 3, 4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bolt-replay-meets-board-validation
    rounds: [2, 3]
    status: resolved
  - key: bolt-empty-commit-readme-stamp
    rounds: [2]
    status: resolved
  - key: pull-first-missing-from-ended-set-callers
    rounds: [2]
    status: resolved
  - key: rewound-rewritten-overlap
    rounds: [3]
    status: resolved
  - key: gone-push-mails-before-done-mark
    rounds: [3]
    status: resolved
  - key: rewritten-way-forward-pull-refuses
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: wake-unreachable-origin-lifts-pause
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: rewound-rewritten-acceptance-split
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 4 raised four findings: one BLOCKING design, two MEDIUM, one LOW.
Key-to-finding map: F1 `rewritten-way-forward-pull-refuses`, F2 `records-lock-waits-unbounded`, F3 `wake-unreachable-origin-lifts-pause`, F4 `rewound-rewritten-acceptance-split`.

Findings fell from 22 to 8 to 7 to 4, and the round-3 BLOCKING keys (Bolt replay validation, busy ended pair) were not raised again, so they are resolved.
`records-lock-waits-unbounded` is now open for a third round: round 3 found the teardown's own waits, round 4 finds the waits the local teardown still makes (hub workspace lock, foreground gc, snapshot locks, per-pair sum).
That recurring enumeration gap makes the round circle.
F1 is new this round and BLOCKING: the rewritten way forward, `lyx fabric pull`, refuses after the reconcile's own fetch.
