# Decision record — one code review perch for darn and loom

## Goal

Darn and loom review the committed diff with one perch, `Code-Review`, under one key set and one rubric, and each recipe sets only the perch's answer key.

## Scope

- The darn recipe's perch rows become `Code-Bouncer`, `Code-Burler-Review` and `Code-Burler-Fix` in segment `Code-Review`, wired as loom's rows are: run subdir `code`, role prefix `code`, carry-over `Code-Review`, loom.yaml's `code_*` fan, model, judge, seed, budget, hard-cap and auto-accept keys.
- darn.yaml loses its perch keys: `darn_reviewer_model`, `darn_fixer_model`, `darn_reviewer_fan`, `darn_fixer_fan`, `review_max_bounces`, `review_hard_max_bounces` and `review_amend_max_bounces`.
- One rubric, `loom-rubric-code-review`, serves both recipes; `loom-rubric-darn-review` is deleted.
- Darn's answer key moves into files: Go writes the task's board entry to a snapshot file in the discussion directory when the darn discussion gate passes, and the perch reads that snapshot and the decision record, never `lyx board get`.
- Docs, PATTERN background text and tests that name the darn perch follow.

## Decisions

### The darn perch reuses loom's row names and segment

Darn's rows take loom's names `Code-Bouncer`, `Code-Burler-Review`, `Code-Burler-Fix` and segment `Code-Review`.
Rationale: the board entry asks for one perch named `Code-Review`; the shared `loomshed` name constants then serve both recipes, and the loom wiring's `Code-Review` segment settings, which `loomSegmentSettings` already builds for every recipe, serve darn with no darn-only segment wiring.
A darn run parked on a retired `Darn-*` perch row is restarted, not migrated, as the recipe header already says for its other retired rows.

### loom.yaml's `code_*` keys are the one key set

The darn perch reads loom.yaml's `code_reviewer_model`, `code_fixer_model`, `code_reviewer_fan`, `code_fixer_fan`, `code_judge`, `code_seed`, `code_max_bounces`, `code_hard_max_bounces`, `code_auto_accept` and the run-wide `review_amend_max_bounces`.
Rationale: darn already reads loom.yaml's `code_judge` and `code_seed`; the darn and loom defaults agree on models, fans and the soft budget of 1.
Darn's perch gains `code_auto_accept`, true by default, and the run-wide hard cap when `code_hard_max_bounces` is 0, in place of darn's own soft-cap rule and hard cap of 2.

### The darn perch keys move to loom.yaml through declared key moves

`configreg.KeyMoves` gains a move from each retired darn key to its loom key: the four model and fan keys to their `code_*` names, `review_max_bounces` to `code_max_bounces`, `review_hard_max_bounces` to `code_hard_max_bounces`, and `review_amend_max_bounces` to `review_amend_max_bounces`.
Rationale: a key move carries a hub's tuned value and a task's or run profile's `run_settings` value into loom.yaml at reconcile and at run-settings preparation, so a darn budget set per run keeps working.
Bound: the moves use the default move rule, so at a base layer a loom value the operator tuned is kept and the darn value dropped, and at a sparse layer any loom value is kept; a darn value the operator tuned now also governs loom's `Code-Review`, which is the one key set the task asks for.

### A darn budget of 0 still skips the perch and Describe

The darn builder keeps its budget-0 rewrite, keyed on the `Code-Review` segment's resolved soft cap: `Darn-Write`'s Done and Finalize's reroute go to `PR-Review`.
Rationale: the writer's description stands when no review changes the code, and loom's own budget-0 behaviour, a Bouncer settling at once, keeps loom's route unchanged.

### One rubric whose answer key the recipe names

`loom-rubric-code-review` names the measuring stick as the answer key files the prompt lists, the Bouncer's artifacts and the review round's answer key, and carries the darn-only and loom-only checks as sections that apply when the answer key holds a board entry snapshot or a plan.
The darn rows name the snapshot and the decision record in `artifact_paths` and in the review profile's `fasit.paths`; loom's rows keep the record and the plan.
Both recipes take the review range from the `review_base` marker, which `Env.ReviewBase` already fills on a darn run.
Rationale: the board entry asks for one rubric with a recipe-set answer key, and the existing `artifact_paths` and `fasit` row config already carry a per-recipe answer key with no new marker.

### Go snapshots the board entry; the record holds only the writer's and parent's additions

On a darn run the `CommitDiscussion` seam first writes the task's board entry, read through `boardengine`, to `_lyx/discussion/board-entry.md`, then commits the discussion directory.
The rubric reads the snapshot as the scope, the record's `Added after Discussion` entries as parent amendments that win over it, and the record's other sections as the writer's own claim, checked against the snapshot and never the answer key.
Rationale: the board entry's open point asks that the copied scope stay apart from the writer's additions; a Go-written file keeps them apart mechanically, and the run's board lock keeps the entry's body and scope fixed for the run, so the snapshot stays true.
Bound: a board read failure fails the commit, which the darn gate already turns into a transient halt rather than a pass over an uncommitted record.

## Constraints

- `PATTERN-recipe-setting-lookup`: the darn rows name their keys through `*_setting` row config, never an own `Env` field.
- `PATTERN-stencil-ownership`: the rubric is read at call time through `stencilstore`.
- `PATTERN-lyxdirs-single-declarer`: the snapshot path is built from `loomengine`'s discussion directory accessor, never a literal `_lyx`.
- `PATTERN-durable-vs-ephemeral-state`: the snapshot is tracked, so it lives under `_lyx`.
- `PATTERN-sole-parsers`: no new parser reads the snapshot; agents read it as prose.
- `PATTERN-test-economy`: renamed rows extend the existing darn recipe, wiring and config tests rather than adding parallel ones.

## Auto-mode assumptions

- The operator proposal's "record file shaped like loom's" is met by darn's existing loom-format decision record plus the separate Go-written snapshot, rather than by the writer copying the board body into the record.
- The darn `pr_review`, writer, verify and tree-scope keys stay in darn.yaml; only the perch's keys move.

## Open risks

- A hub that tuned darn's perch keys and left loom's at their defaults sees its loom `Code-Review` take the darn values after reconcile.
- A darn run in flight on a retired perch row is restarted.

## Acceptance criteria

- The darn recipe builds with `Code-Bouncer`, `Code-Burler-Review` and `Code-Burler-Fix` in segment `Code-Review`, using loom.yaml's `code_*` settings and the `loom-rubric-code-review` rubric.
- A darn budget of 0 in loom.yaml's `code_max_bounces` routes `Darn-Write`'s Done and Finalize's reroute to `PR-Review`; a budget above 0 applies to the perch's rows.
- darn.yaml's template and spec carry no perch key, and a darn.yaml or run_settings document holding a retired darn perch key has it moved to loom.yaml's key.
- The darn `CommitDiscussion` seam writes the board entry snapshot before committing.
- `loom-rubric-darn-review` no longer exists and no code, doc or test names it.

## Notes for the plan writer

- `loomSegmentSettings` in `internal/loomcli/wiring.go` already builds the `Code-Review` segment for darn runs; `wireDarnReview` and `darnReviewSegment` go.
