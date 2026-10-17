# Lens fan and twill fixer for Webster, Merge and Darn review

`code-review-fan` · Task · enhancement, burler, loom, stencil, config · running · PR-Align

## Seen

Every perch runs the same `BurlerReview` and `BurlerFix` engines, wired in one function in `internal/loomcli/wiring.go`.
Only Discussion-Review and Plan-Review can take a lens fan and a twill fixer: `loom.yaml` carries `discussion_fan`, `plan_fan`, `discussion_fix_fan` and `plan_fix_fan`, and `RowClusterFans` and `RowFixFans` hold entries for those rows only.
Webster-Review, Merge-Review and Darn-Review always run solo.
The recorded reason (comments in `contracts/recipes/loom-recipe.yaml` at Webster-Review and Merge-Review) is that fork reviewers may run no git command, so they could not reach a subject that exists only through git.
That ban lives in the fork stencils (`contracts/stencils/burler/burler-cluster-fan.md`, `burler-fix-self-check.md`: "never run any git command").
The engine's fork audit (`internal/burlerengine/cluster.go`) refuses only state-mutating git commands.
The `burler.yaml` fans have lenses for a design record and a plan only, none for code.

## Scope

- Give Webster-Review, Merge-Review and Darn-Review the same fan and fix-fan keys Discussion-Review and Plan-Review have, with the same grammar (one fan name, or a per-round list whose last entry serves every later round; empty runs solo), threaded through the same wiring maps.
  The loom keys live in `loom.yaml`, the Darn-Review keys in `darn.yaml`, since flags live in the config of the module that owns the recipe.
  Template defaults stay empty, so no run changes until a hub sets a key.
- Let fork reviewers and fix-check forks run read-only git (such as `git diff`, `git log`, `git show`), matching the engine's audit, which keeps refusing state-mutating git.
- Remove the recipe comments that say Webster-Review and Merge-Review stay solo, and any doc text resting on that premise.
- Add a code-review fan and a code fix-check fan to the embedded `burler.yaml` template, with lenses for a committed diff (for example correctness, tests, PATTERN fit, leanness), each naming the rubric item it emphasizes without paraphrasing the rubric, as the existing lenses do.
- Update the package `doc.go` files and `docs/overview.md` where they describe which segments can fan.

## Out of scope

- Changing the hub's config; the orch sets the new keys after landing.
- Tuning the lens texts beyond a first draft; later audits of real runs tune them.
