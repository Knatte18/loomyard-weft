---
round: 1
focus:
  - "Issue #58: in `contracts/recipes/motif-recipe.yaml`, `internal/runprofile/template.yaml` and `internal/battenrecipe/names.go`, check whether any path still resolves motif's profile through tapestry, and whether the removed `internal/loomrecipe/recipekeys.go` and `internal/shedbuild/recipe.go` code leaves a caller or a migration (`internal/runprofile/migrate_test.go`) on the old keying."
  - "Issue #53: in `internal/boardengine/layer.go`, `render.go`, `decision.go` and `board.go`, check that an approved-Discussion motif shows as finished in the list, the `--text` listing and the README, and that a `discussed: true` entry which is not approved, or a non-motif entry, is not shown as finished."
  - "Issue #49: in `internal/battencli/commitstatus.go`, `internal/shedengine/run.go` and `internal/loomcli/wiring.go`, check that the board status is written for every batten row past Worktree-Create and then the inner run's current producer; look at failure paths, ordering, and a run whose first row arrives before the inner run starts."
  - "Issue #49 expects `lym loom status` in the task worktree to name the producer in flight. Settle whether the diff changes that path (`internal/loomcli`), and what its tests cover."
  - "Check the `_lym/landing/summary.md` claims against the diff, especially anything it says about what the record's three issues did not need."
  - "Test economy: `internal/loomrecipe/recipekeys_test.go` and `internal/battencli/runsettings_test.go` changed heavily. Check that the removed tests covered only removed behavior, and that each new test in `commitstatus_test.go`, `layer_test.go`, `render_test.go` and `run_commitstatus_test.go` covers behavior no existing test covers."
  - "Check the docs that name the changed behavior (`docs/overview.md`, `contracts/specs/shed-recipe-spec.md`, `plugins/ly/skills/board/SKILL.md`, the `doc.go` files) against the code the diff leaves."
---

Focus is set from the decision record's three issues and the diff stat.
No prior round exists.
