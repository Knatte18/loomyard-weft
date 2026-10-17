---
round: 1
ledger:
  - key: card4-setting-edge-unbounded
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card3-bundles-public-surfaces
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: nil-setting-refusal-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: segment-budgets-way-forward-argument
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: darn-negative-budget-command-lost
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: verify-omits-batten-packages
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: setting-lookup-error-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: config-string-row-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: empty-fixfankey-branch-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card4-impact-summary-readers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings: two MEDIUM and eight LOW, none BLOCKING.
The gating one is the split of card 3, which bundles `shedrecipe.SegmentSettings`, `envkit.TemplateSetting` and the loomcli wiring.
The other MEDIUM, card 4's unbounded `_setting` routing edge, is a consistency finding.
The rest are way-forward, reachability and assertion mismatches in cards 1, 4 and 5, plus a verify gap for the batten packages.
