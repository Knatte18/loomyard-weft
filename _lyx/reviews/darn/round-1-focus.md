---
round: 1
focus:
  - "Config loading for the six new keys (`webster_fan`, `webster_fix_fan`, `merge_fan`, `merge_fix_fan` in loom.yaml; `review_fan`, `review_fix_fan` in darn.yaml): does each get refused as the summary claims for an empty list, a wrong shape and an undefined fan name, with module, key and entry named, and is each closed to the local layer and declared as an open map so reconcile and `--set` carry a list whole?"
  - "The move of `FanList`, its shape error and fan validation from `loomengine` into `burlerengine`, and `KeyAtLine` into `configengine`: do all callers load through the new homes, and does the move leave no behavior change for Discussion and Plan keys?"
  - "Wiring in `wire` and `wireDarnFans`: are the `RowClusterFans`, `RowFixFans` and `RowModelKeys` entries correct per row (Webster, Merge, Darn), including Merge and Darn naming Webster's model keys via `ReviewSettings.WebsterKeys` and the Darn fixer key reading `review_fix_fan in darn.yaml`?"
  - "Fix-half recorded-fan requirement: does the changed refusal for a round reviewed without recorded fans match the summary's stated `goto` way forward, and does a darn row avoid needing recorded fans unless its review row records them?"
  - "Stencil changes (`burler-cluster-fan.md`, `burler-fix-self-check.md`, and the `review-code` and `fix-code` lenses in `burler.yaml`): read against the board entry, does the read-only git allowance drop, weaken or contradict any rule the entry did not ask to change, and do the lens texts name rubric headings that exist?"
  - "Test economy: for the behaviors added, which are covered, which duplicate an existing test, and does any new assertion pin a stencil's static prose?"
  - "Docs and the change description: do `doc.go` files, `docs/overview.md`, config templates and recipe comments agree with the diff, and does every claim in `_lyx/landing/summary.md` hold against the diff?"
---

Scope and test-economy items come first because the diff touches config, wiring and stencils together.
