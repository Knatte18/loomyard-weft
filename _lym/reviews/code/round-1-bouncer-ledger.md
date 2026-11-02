---
round: 1
ledger:
  - key: export-delete-resign-refusal-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: export-name-check-handle-malformed-drift
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parts-gate-worktree-root-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all with local fixes the review text spells out.
F1 maps to `export-delete-resign-refusal-untested`: missing refusal rows for Delete and re-sign exports.
F2 maps to `export-name-check-handle-malformed-drift`: `CheckExportName` doc comment, optionally a shared helper.
F3 maps to `parts-gate-worktree-root-unpinned`: gate tests pass an empty worktree root.
None is gating.
