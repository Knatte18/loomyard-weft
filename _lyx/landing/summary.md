# Darn gets a Darn-Review perch with a darn.yaml budget

The darn recipe keeps its one writer, the `Darn` row, and now runs a review perch on its committed diff before Publish.
The perch is a safety net: it catches what the writer and its verify gate let through, before the operator sees the pull request.

## The perch

`contracts/recipes/darn-recipe.yaml` gains two rows between `Darn` and `Publish`, wired the way loom's `Webster-Review` pair is:

- `Darn-Bouncer` (engine `Bouncer`, segment `Darn-Review`) sends `on_stuck` to `Darn-Burler` and `on_done` to `Publish`.
- `Darn-Burler` (engine `BurlerRound`, same segment) sends `on_stuck` back to `Darn-Bouncer`, runs `fix-scope: source` and commits each fix itself, and runs darn.yaml's verify command as its round gate.

Both rows share `run_subdir: darn`, so their round artifacts land under `_lyx/reviews/darn/`.
A new rubric stencil, `loom-rubric-darn-review`, defines the review: the subject is the branch's diff over its merge base with the run's parent branch, the measuring stick is the task's board entry, and the change description is a claim to check against the diff.
The Bouncer row's required `artifact_paths` names the change description, `_lyx/landing/summary.md`.
There is no `commit_seam` (the fixer commits) and no `carry_over` (a darn run has no decision record).

The perch's judge, seed, reviewer and fixer models are loom.yaml's Webster-Review keys, since both segments review a committed diff.
`internal/loomcli`'s wiring adds `Darn-Bouncer` to `RowBouncerModels` and `Darn-Burler` to `RowReviewModels` on that basis.

`PR-Gate` still bounces to `Darn` on a rejection.
A reworked diff passes through the perch again on its way back to Publish.

## Budget

The new `darn.yaml` key is **`review_max_bounces`**, beside `verify_attempts`.
Its default is **3**, matching loom.yaml's `review_max_bounces`.
It sets `MaxBounces` on both perch rows; a spent budget escalates to the parent the way loom's review segments do.
`0` is valid and turns the review off: `Darn`'s Done routes straight to `Publish`, as before this change.
A value below 0 is refused with a way forward naming `lyx config darn --set review_max_bounces=<n>`, with its row added to `contracts/specs/refusal-spec.md`.
An existing hub's `darn.yaml` without the key resolves to the template default.

## Code

- `internal/loomshed`: new durable row names `NameDarnBouncer` and `NameDarnBurler`, both mapped to `reinvoke` in `InterruptPolicies`.
- `internal/darnengine`: `Config.ReviewMaxBounces` and the template key.
- `internal/shedrecipe`: `Env.DarnReviewMaxBounces`.
- `internal/loomrecipe`: `NewDarn` and `DarnRouting(reviewMaxBounces)` both apply the budget through `applyDarnReviewBudget`, so the status projection reports the routing the engine runs. A budget of 0 rewrites `Darn`'s `OnDone` to `Publish`, and any other value sets the perch rows' `MaxBounces`. `reviewSegments` now takes the recipe source. `BouncerRunSubdir` also searches the darn recipe, so the `lyx loom circling` verbs find the `Darn-Bouncer` run directory.
- `internal/loomcli`: `wireDarn` fills the budget, keeps the loaded `darn.yaml` on the receiver, and replaces `SegmentBounces` with one over the darn routing, so a CIRCLING reason names the darn budget. `recipeRouting` reads the darn budget from the loaded config, or from the hub's `darn.yaml` under the lightweight wiring, where an unreadable file falls back to the template default with a warning, as loom's `reviewBudget` does.

## Resume

Row names are unchanged for every existing row.
A darn run parked at `Darn`, or past it at `Publish`, `PR-Gate` or `Finalize`, resumes on the same rows.
A run parked at `Darn` goes through the perch when `Darn` next returns Done.

## Docs

The darn recipe's header comment, `internal/darnengine` and `internal/loomrecipe` `doc.go`, `docs/overview.md`'s loom module entry and the orch role stencil's darn sentence now describe the perch and its budget.

## Tests

`TestNewDarn_Routing` and `TestDarnRouting` pin the perch's routing, segment and budget, and Darn's route to Publish under a budget of 0.
The darn budget test covers the refusal of a negative budget.
The existing tests for the row table, interrupt policies, `BouncerRunSubdir`, rubric markers, darn config loading and darn wiring now cover the new rows and key.
