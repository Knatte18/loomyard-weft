---
round: 3
ledger:
  - key: fan-shape-error-no-way-forward-or-spec-row
    rounds: [1]
    status: resolved
  - key: bouncer-cluster-fans-ignores-recipe-fan
    rounds: [1]
    status: resolved
  - key: twill-engine-refusal-two-ways-forward
    rounds: [1]
    status: resolved
  - key: unrecorded-names-refusal-archives-before-probe
    rounds: [1]
    status: resolved
  - key: single-fixer-keeps-stale-twill-lenses-and-forks
    rounds: [1]
    status: resolved
  - key: template-self-consistency-test-pins-stale
    rounds: [2]
    status: resolved
  - key: new-lens-emphasis-sentence-unresolved
    rounds: [2]
    status: resolved
  - key: configreg-local-denied-test-omitted
    rounds: [2]
    status: resolved
  - key: fan-shape-error-list-entry-branch-untested
    rounds: [2]
    status: resolved
  - key: fixer-lens-list-sentence-overreach
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: template-header-comment-says-lens-prose-emphasis
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resolve-fan-test-row-duplicates-wantorders
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: load-config-refuses-unknown-fan-rows-unstated
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised nine findings, all resolved by round 3: the review rejected every earlier concern and found no BLOCKING item.
Round 3 raised four new findings, each with a stated local fix.
F1 scopes card 1's lens-text sentence to the reviewer lenses and states the fixer lenses' text separately.
F2 adds the template header comment to card 1's edits.
F3 drops a redundant `TestResolveFan` row in favour of the `wantOrders` map.
F4 states the two existing unknown-fan rows of `TestLoadConfig_Refuses` that card 8's new wording rewrites.
