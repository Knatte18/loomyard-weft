---
round: 1
ledger:
  - key: closed-span-ways-forward-name-refused-verb
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: archived-review-dir-reopens-scope
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pr-agent-operator-exemption-not-built
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: role-files-stencils-still-name-decision-add
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: scope-closed-at-untested-on-real-dir
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: scope-closed-recipe-key-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: hard-cap-answer-route-names-in-scope
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: added-branches-without-test-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: recipe-spec-omits-scope-closed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: decision-add-help-drops-facts
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: plan-spec-tier0-seconds-unconditional
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tier0-budget-key-not-guard-category
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: merge-danger-blast-radius-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.

The main thread is the scope line that closes at the first Code-Review round.
Its refusal leaves other refusals, holds, role files and stencils pointing at the refused verb.
The run-directory read behind the line can reopen after an archive and is untested, as is the `scope_closed` recipe key.

The operator exemption at PR-Review waits on the run parent's ruling.

`tier0-budget-key-not-guard-category` is held at BLOCKING because it cites PATTERN-config-layers.
