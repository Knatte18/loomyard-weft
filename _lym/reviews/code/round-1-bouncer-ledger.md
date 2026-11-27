---
round: 1
ledger:
    - key: batch-upsert-accepts-arras
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: user-visible-text-still-names-arras
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: mechanical
    - key: raddle-operator-notes-in-operatorless-modes
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: split-ready-trigger-line-unnamed
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: dobby-config-comments-stale
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: docs-still-offer-arras-for-board-task
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: child-recipe-read-error-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
---
Round 1 opened every entry above.
The gating one is `batch-upsert-accepts-arras`: the batch write path validates against a snapshot that already holds the incoming entry, so the arras refusal never fires there, and the existing test drives only the single upsert.
The rest are remaining mentions of `arras` in user-visible text and docs, stencil and config wording that disagrees with the new behaviour, and one untested error path.
