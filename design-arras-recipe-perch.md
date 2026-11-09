# The arras recipe and its split perch

`arras-recipe-perch` · Task · feature, shed, loom, burler, arbiter, board · running · Seed-Child

Decision record: [Sketch](decision-arras-recipe-perch.md)

Depends on: [`raddle-perch-mode`](design-raddle-perch-mode.md)

Filed from the Sketch `motif-raddle-recipe`.

Takes: raddle-perch-mode's `perch` mode and outcome checks; the motif and raddle recipes and the generic Bouncer, BurlerReview and BurlerFix engines
Delivers: `contracts/recipes/arras-recipe.yaml` registered wherever motif and raddle are, with its profile blocks and the test pinning `arras` as a former name reversed; the split perch segment with its loom keys, guard-category auto-accept key and the new rubric stencil `loom-rubric-raddle-review.md`; the perch re-arm on a fresh entry into Raddle-Write over a settled verdict; the run-level tests of an arras run through Discussion, the split perch and Raddle-File
Exports: the `arras` recipe name in shedrun, the split perch's loom keys

The part changes these packages:

- contracts/recipes (about 140 changed lines)
- contracts/stencils (about 120 changed lines)
- internal/shedrun (about 15 changed lines)
- internal/loomrecipe (about 90 changed lines)
- internal/loomengine (about 150 changed lines)
- internal/loomcli (about 150 changed lines)
- internal/loomshed (about 40 changed lines)
- internal/shedrecipe (about 30 changed lines)
- internal/shedcli (about 20 changed lines)
- internal/battencli (about 40 changed lines)
- internal/runprofile (about 30 changed lines)
- internal/configsync (about 20 changed lines)
- internal/reviewbudget (about 20 changed lines)
