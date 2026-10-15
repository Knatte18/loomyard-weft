---
round: 1
focus:
  - "contracts/recipes/loom-recipe.yaml: the parent's Webster `on_reroute: Plan-Burler-Review` edge enters a Burler row whose config now carries the task's `role_prefix: plan`; settle whether a reroute into that row, and the sessions it spawns or reattaches to, resolve the same role names the task's prefix scheme expects, and that the row's `on_stuck`/`on_done` cycle still closes after the new edge."
  - "internal/shedadapters/webster.go with internal/shedadapters/bouncer.go and internal/shedrecipe/entries_*.go: the task changed how Bouncer and Burler rows build their session roles and validate config, and the parent added a Stuck Reroute outcome from the Webster row; settle whether the recipe validation and the adapters accept a reroute target that is a Burler row, and whether the task's role derivation is applied on that path."
  - "internal/loomrecipe/recipe_test.go, internal/loomrecipe/bouncerrows_test.go and internal/shedadapters/burlerreview_test.go: both sides edited the recipe's row assertions; settle whether each still asserts what its side meant against the merged `loom-recipe.yaml`, and whether any test is now redundant with another or leaves the reroute-plus-role_prefix interaction uncovered."
  - "internal/websterengine/runlevel.go, internal/websterengine/doc.go and internal/shedadapters/doc.go: settle whether the docs both sides edited agree with the merged behavior, and whether the conflict-record removal in `Run` interacts with anything the task changed in the Webster segment's rows."
  - "docs/overview.md and the Webster-Review segment rows of the recipe: the task touched the stage and recipe description while the parent added a reroute edge; settle whether the merged overview and recipe header comments still describe the segment wiring correctly."
---

The seed is a reading of the two ranges' overlap: the parent added a Webster-to-Plan-Burler-Review reroute and conflict-record routing, and the task added `role_prefix` across every review segment's rows.
The task's checklist and batten overview commits touch no file the parent changed, so they are not in the focus.
