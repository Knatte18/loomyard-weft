---
round: 1
ledger:
  - key: seed-parent-param-name-duplicated
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: seed-read-refusal-message-unasserted
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: shed-recipe-spec-states-old-rules
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: record-seed-wiring-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: merge-danger-understated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: arras-header-points-at-tapestry-ring-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loomshed-comments-place-rows-in-tapestry
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: jacquard-comment-names-absent-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stored-only-support-log-misstates-source
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: nested-run-ending-block-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stored-only-cancel-check-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-names-task-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round.
The review finds that the diff makes the change the amended board entry asks for.
The open items are two PATTERN breaches (a repeated seed-param name and an unasserted refusal message), test gaps around Record-Seed and nested blocks,
one false support-log claim in the stored-only mode, and stale text in the spec, comments, recipe headers and change description.
