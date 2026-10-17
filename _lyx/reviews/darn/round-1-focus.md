---
round: 1
focus:
  - "Migrate hooks in `internal/loomengine/config.go` and `internal/darnengine/config.go`: does every retired key map to the right new key, including the both-names case and the Merge copy from `webster_review`/`webster_fix`, and is the same hook applied to `run_settings`?"
  - "Model-key resolution in `internal/loomengine/review.go` and `internal/loomcli/darnwiring.go`: does an empty segment key (Merge and Darn included) fall back to the shared `reviewer_model`/`fixer_model`, and does an empty list `[]` behave as the change description claims?"
  - "Refusal and engine-error messages: does each name the key the model actually came from (`shuttleengine.ModelKey` module and key), for both loom.yaml and darn.yaml keys?"
  - "Leftover old names: any remaining `review`, `fix`, `<segment>_review`, `<segment>_fix`, `*_fix_fan`, `review_fan` model or fan key in templates, `ConfigSpec` categories, recipe comments, docs, `doc.go` files and help text, against the board entry's rename list."
  - "Scope: do the non-model `review_*` knobs and the other role keys (`judge`, `seed`, `rework` and the rest) keep their names, and does the diff change anything beyond the board entry's ask?"
  - "The change description's claims against the diff, notably the Merge-Review and Darn-Review default models staying equal to Webster's, and the acknowledged unmet acceptance criterion on strict loads of old keys."
  - "Repository rules: `PATTERN.md` invariants, `docs/overview.md` and `doc.go` updates in the same diff, the `cmd/lyx/testdata/test-budget.yaml` row against the new top-level tests, and test economy (duplicated coverage, uncovered new behavior)."
---

Seeded from the board entry, the change description and the diff stat; no prior round exists.
