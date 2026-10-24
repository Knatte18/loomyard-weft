# One Code-Review perch for darn and loom

Darn and loom reviewed the committed diff with two perches that ran the same `BurlerReview` engine under separate keys and rubrics.
The two now share one perch, `Code-Review`, with one key set and one rubric, and each recipe sets only the answer key the perch reads.

## What changes

- The darn recipe's perch rows are `Code-Bouncer`, `Code-Burler-Review` and `Code-Burler-Fix` in segment `Code-Review`, wired as loom's rows are.
  The `Darn-*` perch rows, `wireDarnReview` and `darnReviewSegment` are gone; `loomSegmentSettings` already builds the segment for every recipe.
- The darn perch reads loom.yaml's `code_*` keys and the run-wide `review_amend_max_bounces`.
  It gains `code_auto_accept`, and the run-wide hard cap when `code_hard_max_bounces` is 0, in place of darn's own soft-cap rule and hard cap of 2.
- darn.yaml's template and spec lose `darn_reviewer_model`, `darn_fixer_model`, `darn_reviewer_fan`, `darn_fixer_fan`, `review_max_bounces`, `review_hard_max_bounces` and `review_amend_max_bounces`.
- `loom-rubric-code-review` serves both recipes and `loom-rubric-darn-review` is deleted.
  The rubric treats the answer key files the prompt lists as the measuring stick, and its darn-only and loom-only checks apply when that key holds a board entry snapshot or a plan.
- On a darn run the `CommitDiscussion` seam first writes the task's board entry, read through `boardengine`, to `_lyx/discussion/board-entry.md` (`darnengine.BoardEntrySnapshot`), then commits the discussion directory.
  The perch reads the snapshot as the scope, the record's `Added after Discussion` entries as amendments that win over it, and the record's other sections as the writer's own claim.
- Docs, `PATTERN-verified-tree.md` and tests that named the darn perch follow.

## Design decisions

- **Key moves.**
  `configreg.KeyMoves` moves each retired darn key to its loom key, including the former `review_fan` and `review_fix_fan` names, so a hub's tuned value and a task's or run profile's `run_settings` value carry over at reconcile and at run-settings preparation.
  The default move rule applies: at a base layer a loom value the operator tuned is kept and the darn value dropped, and at a sparse layer any loom value is kept.
  A hub that tuned darn's perch keys and left loom's at their defaults sees its loom `Code-Review` take the darn values.
- **Former defaults are dropped.**
  `configsync` gains `formerSourceDefaults`: a base-layer source value equal to the template value the key held before the move was never tuned, so the move drops it with a report line instead of overwriting the target's default.
  Only values that differ from the target's current default are listed.
  A file that carries only such report lines now gets a `Result`, which is never `Applied`.
- **Budget 0 still skips the perch and Describe.**
  The darn builder keeps its budget-0 rewrite, now keyed on the `Code-Review` segment's resolved soft cap, so `Darn-Write`'s Done and Finalize's reroute go to `PR-Review`.
  `rerouteReviewed` and `codeReviewOff` share `codeBudgetZero`, which reads loom.yaml for both recipes.
- **Snapshot failure halts.**
  A board read failure fails the commit, which the darn gate already turns into a transient halt rather than a pass over an uncommitted record.
  The snapshot is tracked under `_lyx`, built from `loomengine`'s discussion directory accessor, and read by agents as prose, so no new parser reads it.
- **Retired rows restart.**
  A darn run parked on a retired `Darn-*` perch row is restarted, not migrated, as the recipe header says for its other retired rows.

## Not run by hand

- A darn run end to end through `Code-Review` against a real hub, including the budget-0 route to `PR-Review`.
- `lyx config` reconcile on a hub whose darn.yaml holds tuned perch keys, to see the moved values and report lines.
