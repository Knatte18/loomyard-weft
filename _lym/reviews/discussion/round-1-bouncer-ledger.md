---
round: 1
ledger:
  - key: board-scope-lens-prose-rule-conflict
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-speed-loom-fan-pick-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d5-loom-retired-fan-keys-route-unnamed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-edit-sites-hold-no-such-text
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-risk-2-deleted-lens-failure-reach
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-template-value-vs-hub-value
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, none recurring.
F1 (`board-scope-lens-prose-rule-conflict`) is the only gating one: D1 moves `board-scope` out of the `review-*` and `fix-*` fans, so the template's procedure-lens rule and `TestConfigTemplate` no longer exempt it from the emphasis line.
F2 and F3 are LOW design gaps in D6 and D5; F4 to F6 are LOW or NIT text corrections in D7, Open risks and D6.
