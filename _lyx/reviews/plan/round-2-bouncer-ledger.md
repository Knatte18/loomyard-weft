---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card1-unregistered-module-key-assertion
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-one-perch-table-claim
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shed-recipe-spec-env-rule-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card1-configfile-base-dir
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised ten findings, two MEDIUM and eight LOW.
Round 2 does not re-raise any of them and reports no regression from round 1's fixes, so all ten are resolved.

Round 2 raised five new findings.
The gating one is BLOCKING: card 6 inserts the declared `PATTERN-recipe-setting-lookup` line and leaves `PATTERN.md` over the headroom line, so the card gate and every later verify gate refuse the branch until other overview lines are shortened.
The other four are local text corrections in cards 1, 3 and 5: an assertion that names a key the message lacks, a perch-table comment that overclaims, a spec sentence that keeps the replaced `Env` rule, and the wrong base directory for `ConfigFile`.
