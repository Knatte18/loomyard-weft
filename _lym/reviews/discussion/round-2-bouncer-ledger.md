---
round: 2
ledger:
  - key: d8-reinterpretation-after-retired-prune
    rounds: [1]
    status: resolved
  - key: d1-gaps-under-current-never-requeued
    rounds: [1]
    status: resolved
  - key: d8-off-switch-lost-in-worktree-local-layers
    rounds: [1]
    status: resolved
  - key: d8-d9-one-slot-restart-loop
    rounds: [1]
    status: resolved
  - key: d9-yield-lock-hand-call-mutual-wait
    rounds: [1]
    status: resolved
  - key: d10-parallel-rule-selects-nothing
    rounds: [1]
    status: resolved
  - key: d11-wrong-actor-anchor-verb
    rounds: [1]
    status: resolved
  - key: d8-d9-pass-vs-hand-call-marker
    rounds: [1]
    status: resolved
  - key: d2-d3-testdata-criterion-contradiction
    rounds: [1]
    status: resolved
  - key: open-risk-cross-package-run-at-darn-write
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-env-yaml-outside-stamp
    rounds: [1]
    status: resolved
  - key: d1-d8-lock-skipped-map-not-queued
    rounds: [1]
    status: resolved
  - key: d7-self-check-names-absent-table
    rounds: [1]
    status: resolved
  - key: d5-d8-d12-carve-out-and-queued-count
    rounds: [1]
    status: resolved
  - key: d13-carrier-skips-notes
    rounds: [1]
    status: resolved
  - key: d8-off-switch-before-walk
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-d10-whole-rebuild-write-keeps-stale-entries
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-failed-self-registration-escapes-d11
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d9-lock-retake-blocks-while-holding-slot
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d1-d4-keep-rule-vs-outcome-entry-drop
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-test-map-upkeep-category-unstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d8-first-criterion-lacks-slot-condition
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-d6-rebuild-never-writes-current-stamp
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 15 findings, two BLOCKING.
Round 2 raised 9, none BLOCKING; round 1's fourteen design, consistency and LOW keys other than the cross-package one are not re-raised and are carried as resolved.
The cross-package Open risks line stays open only as a LOW `consistency` finding, since the parent's `Added after Discussion` entry settles it and the fix drops the line.
The two MEDIUM design findings are new keys: the D8 off switch before any reconcile walk, and the stale entries a whole rebuild's first write may carry to H.
The remaining new findings are LOW: failed self-registration, the D9 lock retake, D1/D4 wording, the new key's category, the D8 criterion's slot condition and the rebuild's environment stamp.
