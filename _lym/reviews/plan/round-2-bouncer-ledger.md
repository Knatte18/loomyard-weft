---
round: 2
ledger:
  - key: parts-max-env-field
    rounds: [1]
    status: resolved
  - key: plan-prompt-reorder-vs-parent-condition
    rounds: [1]
    status: resolved
  - key: chair-edits-parts-file-and-part-export-missing
    rounds: [1]
    status: resolved
  - key: plan-prompt-golden-stencil-prose
    rounds: [1]
    status: resolved
  - key: card-format-pointer-docs-stale
    rounds: [1]
    status: resolved
  - key: plan-spec-mode-list-misses-part-check
    rounds: [1]
    status: resolved
  - key: reportengine-doc-plan-phase-rows
    rounds: [1]
    status: resolved
  - key: planglyph-doc-part-scope-exemptions
    rounds: [1]
    status: resolved
  - key: seatengine-doc-writer-seat-sentences
    rounds: [1]
    status: resolved
  - key: joined-shared-decisions-stale-card-numbers
    rounds: [1]
    status: resolved
  - key: join-unmet-export-unreachable-via-verb
    rounds: [1]
    status: resolved
  - key: validate-plan-part-test-repeats-validate-part
    rounds: [1]
    status: resolved
  - key: validate-plan-part-no-parts-row-check
    rounds: [1]
    status: resolved
  - key: card7-spec-extends-should-be-edit
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card5-part-scope-resolve-rules-no-spec
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card9-stepender-comments-stale
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card18-validate-plan-help-part-claim
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: part-check-drops-pattern-line-unprescribed
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: parts-plan-table-asserts-block-prose
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card9-writer-variants-repeat-advisor-rows
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card18-verb-tests-repeat-rows
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card12-step3-blank-lines-omitted
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 13 findings, one BLOCKING; none recurs in round 2, so all 13 keys are resolved.
Round 2 raised nine new findings (F1 to F9 in the review's order, mapped to the keys above).
F1 is the only MEDIUM and is consistency-class; the design findings F6 to F8 are LOW test-economy trims.
F5 needs a parent-recorded decision entry, hence `design-wrong`.
