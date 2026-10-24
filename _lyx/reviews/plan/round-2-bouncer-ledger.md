---
round: 2
ledger:
  - key: inert-share-computescope-switch
    rounds: [1]
    status: resolved
  - key: csharp-seed-tree-unreachable-scopes
    rounds: [1]
    status: resolved
  - key: dotnet-guards-refusal-config-way-forward
    rounds: [1]
    status: resolved
  - key: card6-failed-git-read-two-ways
    rounds: [1]
    status: resolved
  - key: build-cannot-start-no-refusal
    rounds: [1]
    status: resolved
  - key: build-timeout-reason-names-test
    rounds: [1]
    status: resolved
  - key: blocked-tier-record-no-build-cause
    rounds: [1]
    status: resolved
  - key: dotnet-narrowing-no-stated-bound
    rounds: [1]
    status: resolved
  - key: partial-set-on-whole-run
    rounds: [1]
    status: resolved
  - key: gate-test-help-misses-build-step
    rounds: [1]
    status: resolved
  - key: config-scoper-comment-stale
    rounds: [1]
    status: resolved
  - key: filter-percent-way-forward-caller-words
    rounds: [1]
    status: resolved
  - key: filter-budget-measures-terms
    rounds: [1]
    status: resolved
  - key: filter-percent-refusal-no-reaching-test
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: csharp-seed-classes-not-test-classes
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: impactset-reads-bundled-in-entry-point-card
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised 13 findings; round 2 re-raised none of them, so all are resolved.
Round 2 raised three LOW design findings on new keys.
F1 is the card 7 percent-sign refusal that no test reaches on a non-Windows host.
F2 is card 8's seed classes, not stated to carry test attributes.
F3 is card 6's bundling of the two exported `impactset` reads, which would be split out with `insert-card`.
