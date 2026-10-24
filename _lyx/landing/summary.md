# One Code-Review perch for darn and loom

Darn and loom each reviewed the committed diff with their own perch over the same `BurlerReview` engine: darn's `Darn-Review` (`Darn-Bouncer`, `Darn-Burler-Review`, `Darn-Burler-Fix`, darn.yaml's `darn_*` and `review_*` keys, rubric `loom-rubric-darn-review`) and loom's `Code-Review`.
This change merges them into one perch, `Code-Review`, with one key set and one rubric; each recipe sets only the perch's answer key.

## What changes

- **Recipe.** `contracts/recipes/darn-recipe.yaml` now runs `Code-Bouncer`, `Code-Burler-Review` and `Code-Burler-Fix` in segment `Code-Review`, with loom's run subdir `code`, role prefix `code`, carry-over segment, rubric and `loom.code_*` setting keys, `code_auto_accept` included.
  `Darn-Write`'s Done and Finalize's reroute point at `Code-Bouncer`.
  The `loomshed` `NameDarnBouncer`, `NameDarnBurlerReview` and `NameDarnBurlerFix` constants are gone; both recipes use the `NameCode*` ones.
- **Answer key.** On a darn run the perch reads `_lyx/discussion/board-entry.md` and the decision record, both in the Bouncer's `artifact_paths` and the review profile's `fasit.paths`.
  The darn `CommitDiscussion` seam (`commitDarnDiscussion` in `internal/loomcli/darnwiring.go`) writes that snapshot of the task's board entry, rendered by the new `darnengine.BoardEntrySnapshot`, before it commits the discussion directory.
  The board entry's own text thereby stays apart from the writer's record: the record's `Added after Discussion` entries amend the snapshot, and its other sections are the writer's claim, checked against it.
  A board read failure, or no entry for the task, fails the commit, which the darn gate already turns into a halt rather than a pass.
- **Rubric.** `loom-rubric-code-review` serves both recipes and names the measuring stick as the answer key the prompt lists, in one of two forms: a plan with its decision record, or a board entry snapshot with its record.
  The checks only one form needs (per-card mechanical checks; unrecorded parent corrections and own-entry steps) say which form they apply to.
  Both recipes take the review range from `{{.review_base}}`, which `Env.ReviewBase` already filled on a darn run.
  `loom-rubric-darn-review` is deleted.
- **Config.** darn.yaml loses `darn_reviewer_model`, `darn_fixer_model`, `darn_reviewer_fan`, `darn_fixer_fan`, `review_max_bounces`, `review_hard_max_bounces` and `review_amend_max_bounces`, with `MigrateConfig`, the model, fan and budget validation and the budget part of `ValidateLocalValues`.
  `configreg.KeyMoves` declares a move from each of them, and from the older `review_fan` and `review_fix_fan`, to loom.yaml's `code_*` key, or to the run-wide `review_amend_max_bounces`.
  Reconcile and run-settings preparation apply the moves, so a tuned hub value and a task's or run profile's `run_settings` budget carry over.
  The moves use the default move rule: at a base layer a tuned loom value is kept, and at a sparse layer any loom value is kept.
- **Wiring.** `wireDarnReview`, `ResolveDarnReview`, `darnReviewBudget` and the `darnCfg` field are gone: `loomSegmentSettings` already builds the `Code-Review` segment for every recipe, and the darn routing reads loom's segment budgets.
  `rerouteReviewed` reads the Code-Perch's soft cap for both recipes, and `perchHardCap` reads loom.yaml for every Bouncer row.
- **Budget 0.** A `code_max_bounces` of 0 still routes a darn run's `Darn-Write` Done and Finalize reroute straight to `PR-Review`, past the perch and `Describe`; loom's own budget-0 behaviour is unchanged.
- **Reports.** `reportengine` keeps the retired `Darn-*` perch rows as literals mapped to the review phase, so past darn runs still time correctly.

## Design decisions

- The darn perch reuses loom's row names rather than new shared ones, so the existing `NameCode*` constants and the loom wiring's `Code-Review` segment serve darn unchanged.
  A darn run parked on a retired `Darn-*` perch row is restarted, not migrated: its run subdir changes from `darn` to `code`, so a renamed row would find none of its round artifacts.
- The darn perch adopts loom's defaults: `code_auto_accept` true and the run-wide hard cap when `code_hard_max_bounces` is 0, in place of darn's soft-cap rule and its hard cap of 2.
- The board entry snapshot is Go-written rather than copied into the record by the writer, so the copy cannot drift from the entry and needs no discussion format change.
- The escalation test of the darn perch now runs with `code_auto_accept` off, since that is the path it pins.

## Docs

`docs/overview.md`, the `darnengine`, `loomrecipe`, `loomshed`, `loomcli`, `reportengine` and `configcli` package docs, `pattern/PATTERN-verified-tree.md` and the orch role stencil follow the rename and the moved keys.
