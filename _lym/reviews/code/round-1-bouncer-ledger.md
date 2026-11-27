---
round: 1
ledger:
    - key: dirty-worktree-row-vacuous-merge-base
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: summary-wrong-plan-directory
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: scenariohub-comment-overstates-hub-rows
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
---
Round 1 opened every key.

- `dirty-worktree-row-vacuous-merge-base`:
  with a fresh hub per row, the whole-green commit the row records was the merge base itself,
  so its assertion could not tell the two apart.
- `summary-wrong-plan-directory`:
  the change description placed the plan under `.lym`, while it sits under `_lym/plan`.
- `scenariohub-comment-overstates-hub-rows`:
  the type comment said every row builds a hub, while some rows build none.

The review rejected one candidate, about measurements taken without `--gate darn-write`.
