---
round: 1
ledger:
    - key: resumed-recovery-loses-question-and-answer
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: every-message-turn-end-becomes-waiting
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: external-test-sibling-listing-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: last-turn-message-error-leaves-strand-live
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: last-turn-message-offset-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: created-member-sibling-branches-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: rearm-reaches-failed-and-stuck-batches
      rounds:
        - 1
      status: open
      class: scope
      severity: LOW
      fix: design-wrong
    - key: amended-cards-docs-stale
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: plan-review-rubric-unframed-run-card-index
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: merge-danger-blast-radius-too-narrow
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
---
Round 1 is the first round, so every key is new and open.

The weight is on the recovery wait path in `internal/websterengine`:
a resumed recovery gets neither its question nor the parent's answer and the asking loop is unbounded,
and any turn end with a message is routed to the parent as a question.
The one other gating item is a missing regression row for the external-test sibling listing in `internal/planglyph`.

The LOW findings cover a strand left live on an error path, test gaps, the breadth of the recovery re-arm,
and stale or understated text in docs, a rubric stencil and the change description.
