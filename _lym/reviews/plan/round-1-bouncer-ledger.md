---
round: 1
ledger:
  - key: parts-max-env-field
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: plan-prompt-reorder-vs-parent-condition
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: chair-edits-parts-file-and-part-export-missing
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: plan-prompt-golden-stencil-prose
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card-format-pointer-docs-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: plan-spec-mode-list-misses-part-check
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: reportengine-doc-plan-phase-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: planglyph-doc-part-scope-exemptions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seatengine-doc-writer-seat-sentences
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: joined-shared-decisions-stale-card-numbers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: join-unmet-export-unreachable-via-verb
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: validate-plan-part-test-repeats-validate-part
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: validate-plan-part-no-parts-row-check
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 13 findings (F1 to F13 in the review's order, mapped to the keys above).
F1 is the one BLOCKING finding: card 16's `PartsMax` field on `shedrecipe.Env` breaks `PATTERN-shed-recipe-registry`.
F2 and F3 ask for a parent ruling on decisions the plan departs from; F4 and F12 are test-economy; F5 to F10 and F13 are stale-doc and consistency fixes; F11 depends on how F3 is settled.
Nothing is resolved yet.
