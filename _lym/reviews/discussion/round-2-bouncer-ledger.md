---
round: 2
ledger:
  - key: board-scope-lens-prose-rule-conflict
    rounds: [1]
    status: resolved
  - key: d6-speed-loom-fan-pick-unstated
    rounds: [1]
    status: resolved
  - key: d5-loom-retired-fan-keys-route-unnamed
    rounds: [1]
    status: resolved
  - key: d7-edit-sites-hold-no-such-text
    rounds: [1]
    status: resolved
  - key: open-risk-2-deleted-lens-failure-reach
    rounds: [1]
    status: resolved
  - key: d6-template-value-vs-hub-value
    rounds: [1]
    status: resolved
  - key: d2-way-forward-loses-emphasis-exemption
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d5-local-layer-bound-omits-fan-swap
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: auto-mode-omits-d1-review-discussion-pick
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-migrate-route-hand-edited-local
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's six findings did not recur in round 2, so all are resolved.
Round 2 raised four new findings.
F1 (`d2-way-forward-loses-emphasis-exemption`) is the only gating one: D2 takes `way-forward` out of `review-plan`, so `TestConfigTemplate` requires it to end with the emphasis line.
F2 to F4 are a LOW bound gap in D5, a LOW omission in Auto-mode assumptions and a NIT on which routes run Migrate.
Each fix is a local text correction the finding already spells out.
