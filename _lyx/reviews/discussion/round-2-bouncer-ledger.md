---
round: 2
ledger:
  - key: bolt-replay-layout-premise
    rounds: [1]
    status: resolved
  - key: ended-means-removed-from-origin
    rounds: [1]
    status: resolved
  - key: wake-paused-run-not-moved
    rounds: [1]
    status: resolved
  - key: ended-set-outside-reconcile
    rounds: [1]
    status: resolved
  - key: cleanup-dry-run-mutates
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: push-deferral-loops-or-silent
    rounds: [1]
    status: resolved
  - key: own-run-exclusion-resume-unreachable
    rounds: [1]
    status: resolved
  - key: ended-busy-gets-resume-way-forward
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
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
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: records-lock-waits-unbounded
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: bolt-replay-meets-board-validation
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: bolt-empty-commit-readme-stamp
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pull-first-missing-from-ended-set-callers
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 2 raised eight findings: four BLOCKING design, two MEDIUM design, two LOW consistency.
Key-to-finding map: F1 `wake-reconcile-held-by-own-hub-pause`, F2 `records-lock-waits-unbounded`, F3 `cleanup-dry-run-mutates`, F4 `bolt-replay-meets-board-validation`, F5 `bolt-empty-commit-readme-stamp`, F6 `pull-first-missing-from-ended-set-callers`, F7 `gone-class-source-stale`, F8 `ended-busy-gets-resume-way-forward`.

Three round 1 keys reopened: the cleanup dry run (its fetch moves the tracking refs the rewound judgment reads), the gone class (its ended-set source is not refreshed), and the ended busy pair (its new way forward does not end the driver).
The other round 1 keys were not raised again and are resolved.
`prime-lock-wait-unbounded` is resolved because the prime lock is now a single try-acquire; the records-branch locks carry the same defect under the new key `records-lock-waits-unbounded`.

The remaining design problems cluster around the wake path under a hub pause, the lock bounds on records branches, the evidence for rewound branches, and the Bolt replay's meeting with the board's write validation.
