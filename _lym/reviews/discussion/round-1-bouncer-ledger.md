---
round: 1
ledger:
  - key: d8-reinterpretation-after-retired-prune
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d1-gaps-under-current-never-requeued
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d8-off-switch-lost-in-worktree-local-layers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-d9-one-slot-restart-loop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-yield-lock-hand-call-mutual-wait
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d10-parallel-rule-selects-nothing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d11-wrong-actor-anchor-verb
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-d9-pass-vs-hand-call-marker
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-d3-testdata-criterion-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: open-risk-cross-package-run-at-darn-write
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d6-env-yaml-outside-stamp
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d1-d8-lock-skipped-map-not-queued
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-self-check-names-absent-table
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-d8-d12-carve-out-and-queued-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d13-carrier-skips-notes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 15 findings, none carried from earlier rounds.
The two BLOCKING design findings are the D8 migration order (the retired-key prune runs before reinterpretations, so `Module.Migrate` is the mechanism) and D1's requeue rule for gaps left under a Current verdict.
Six further design findings concern D8, D9, D10 and D11 mechanics; F10 is a parent question about the running `testmap-cross-package` entry.
The rest are consistency and LOW corrections to D5 to D8, D12 and D13.
