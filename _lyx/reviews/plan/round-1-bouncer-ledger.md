---
round: 1
ledger:
  - key: inert-share-computescope-switch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: csharp-seed-tree-unreachable-scopes
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: dotnet-guards-refusal-config-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card6-failed-git-read-two-ways
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: build-cannot-start-no-refusal
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: build-timeout-reason-names-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: blocked-tier-record-no-build-cause
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
  - key: dotnet-narrowing-no-stated-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: partial-set-on-whole-run
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-test-help-misses-build-step
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: config-scoper-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: filter-percent-way-forward-caller-words
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: filter-budget-measures-terms
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 13 findings, none recurring.
The gating one is F1: card 10's inert-only share does not say how it passes `computeScope`'s closing switch under `go` and `none`.
F2 (C# seed tree) is the other MEDIUM, class `consistency`.
F7 is the one fix that adds a `RunRecord` field, so it counts as changing a contract.
