---
round: 3
ledger:
  - key: card4-setting-edge-unbounded
    rounds: [1]
    status: resolved
  - key: card3-bundles-public-surfaces
    rounds: [1]
    status: resolved
  - key: nil-setting-refusal-way-forward
    rounds: [1]
    status: resolved
  - key: segment-budgets-way-forward-argument
    rounds: [1]
    status: resolved
  - key: darn-negative-budget-command-lost
    rounds: [1]
    status: resolved
  - key: verify-omits-batten-packages
    rounds: [1]
    status: resolved
  - key: setting-lookup-error-assertion
    rounds: [1]
    status: resolved
  - key: config-string-row-assertion
    rounds: [1]
    status: resolved
  - key: empty-fixfankey-branch-unreachable
    rounds: [1]
    status: resolved
  - key: card4-impact-summary-readers
    rounds: [1]
    status: resolved
  - key: card6-pattern-headroom-overflow
    rounds: [2]
    status: resolved
  - key: card1-unregistered-module-key-assertion
    rounds: [2]
    status: resolved
  - key: card5-one-perch-table-claim
    rounds: [2]
    status: resolved
  - key: shed-recipe-spec-env-rule-stale
    rounds: [2]
    status: resolved
  - key: card1-configfile-base-dir
    rounds: [2]
    status: resolved
  - key: card5-fan-setting-wiring
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card6-lookup-error-names-template
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised fifteen findings; none recurs in round 3, so all are resolved.
Round 3 raised two LOW consistency findings, both local text corrections: card 5's Setting assertion names a wiring that seeds no fan key (with card 8 then dropping the only own-key coverage), and card 6 claims every lookup error names the template.
