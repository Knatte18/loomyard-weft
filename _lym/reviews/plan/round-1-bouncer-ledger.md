---
round: 1
ledger:
  - key: raddle-name-declared-twice
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: plain-promote-way-forward-not-raddle
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: project-type-label-unsourced
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: unknown-decision-pattern-id-unchecked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: store-slug-bound-overstates-start-guard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: raddle-stencil-omits-parent-edit-directives
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: arbiter-recipe-description-raddle
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: filing-level-departs-from-parent-constraint
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: filed-task-body-unrendered
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: loomcli-doc-go-left-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: status-draft-comment-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: docs-overview-misses-config-keys
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: legacy-sketch-label-tasks-unwritable
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: discussed-promotion-replay-bypass-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: batten-guard-pass-branch-rationale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: handoff-post-result-dropped
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: slug-sketch-never-fires-alone
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card-4-research-steps
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: session-end-nil-hook-row-redundant
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: amendment-target-comment-missed
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twenty findings, one per key above, in review order F1 to F20.
The gating ones are the BLOCKING duplicate declaration of the recipe name and the two MEDIUM design findings on the unchecked decision and PATTERN ids and on the overstated store-slug Bound.
The design-wrong keys change a decision, a check set or a behavior and need a fresh review of their new text; the rest are local corrections.
No relabel was applied.
