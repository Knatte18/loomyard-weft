---
round: 1
ledger:
  - key: bolt-replay-layout-premise
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: ended-means-removed-from-origin
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: wake-paused-run-not-moved
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: ended-set-outside-reconcile
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: cleanup-dry-run-mutates
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: prime-lock-wait-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fabricengine-seams-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: notice-lost-without-orch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: set-aside-tags-collide-with-archive
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: gone-class-source-stale
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: push-deferral-loops-or-silent
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: own-run-exclusion-resume-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: ended-busy-gets-resume-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: code-branch-rewrite-misread
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: destruction-chokepoint-covers-tags
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: finalize-parent-lock-order
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-bound-ignores-set-aside
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: notice-signer-not-always-fabric
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: bolt-two-push-locks-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gone-ended-weft-false-alarm
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: q8-contradicts-d8
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: goal-cites-wrong-section
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review, so no earlier ledger exists and every entry is open.
Key-to-finding map: F1 `bolt-replay-layout-premise`, F2 `ended-means-removed-from-origin`, F3 `wake-paused-run-not-moved`, F4 `ended-set-outside-reconcile`, F5 `cleanup-dry-run-mutates`, F6 `prime-lock-wait-unbounded`, F7 `fabricengine-seams-missing`, F8 `notice-lost-without-orch`, F9 `set-aside-tags-collide-with-archive`, F10 `gone-class-source-stale`, F11 `push-deferral-loops-or-silent`, F12 `own-run-exclusion-resume-unreachable`, F13 `ended-busy-gets-resume-way-forward`, F14 `code-branch-rewrite-misread`, F15 to F22 in order from `destruction-chokepoint-covers-tags` to `goal-cites-wrong-section`.

The gating cluster is the Bolt replay layout, the definition of an ended entry, the wake path for paused runs, the ended set's reach outside the reconcile, and the cleanup dry run.
Most of the rest are bounds and seams the record leaves unstated.
