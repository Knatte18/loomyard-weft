## Goal

Replace the hub's ad hoc run profiles (`budget`, `light`, `docs`, `darn-wide`) with one ladder, `speed`, `light`, `solid` and `wide`, that darn and loom share by name, each profile holding one settings block per recipe, plus a per-recipe default profile.

## Scope

- `internal/runprofile`: the profiles document's shape becomes profile name → recipe → module → keys, with a reserved top-level `defaults` entry mapping a recipe to its default profile name; the template seeds the ladder, `docs` and the defaults.
- `internal/battenshed` and `internal/battencli`: Seed-Child resolves the profile for the task's recipe (the task's own profile, else the recipe's default) and lays the task's `run_settings` over that recipe's block only.
- `internal/configreg`: the profiles check validates each recipe block of each profile as run settings.
- `internal/boardcli`: `lyx board profiles` prints the defaults beside the profiles; upsert still checks only that a named profile exists.
- Docs: `internal/runprofile/doc.go`, `internal/battenshed/doc.go`, `internal/boardengine/doc.go`, `internal/configcli/doc.go` and `docs/overview.md` where they describe the profile shape.
- Out of scope: rewriting the hub's own `profiles.yaml` on Bolt, which the parent does from the hub's prime worktree after the binary deploys.

## Decisions

- **A profile holds one block per recipe.**
  Each block has the shape of a board task's `run_settings`, module name to keys, and Seed-Child writes only the block of the recipe the task runs.
  Rationale: darn's Code-Review perch reads loom's `code_*` keys, so the Code and Darn ladder columns need the same key to take a different value per recipe; the operator decided this form on 2026-10-24.
- **The ladder values.**
  Every perch's hard cap is its rounds plus one.
  `speed`: loom Discussion 0, Plan 0, Code 1; darn Code 0.
  `light`: loom 1, 1, 1; darn 1.
  `solid`: loom 2, 2, 1; darn 2.
  `wide`: loom 3, 2, 1; darn 3.
  Rationale: the operator's table, with the hard cap the operator asked for so `lyx loom circling continue` can grant one more round.
- **`docs` is a darn-only profile.**
  Its darn block sets `darn.writer: sonnet[medium]` and one Code perch round.
  Rationale: the operator kept it for pure prose and config value edits.
- **A round count of 0 turns a perch off through profile settings alone.**
  The `<perch>_max_bounces` keys already document that 0 settles the perch's Bouncer approved with no review round, so no entry gate is needed.
- **The default profile per recipe lives in `profiles.yaml` under a reserved `defaults` entry.**
  `defaults: {loom: solid, darn: light}` in the template; a task with an empty `profile` takes its recipe's default, and a recipe without one runs on the hub's config as before.
  A profile cannot be named `defaults`, and a default naming no profile, or a profile with no block for that recipe, is refused at load.
  Rationale: the default names a profile, so it belongs with the profiles; a recipe module key would also be settable through `run_settings`, where it means nothing.
- **A profile with no block for the task's recipe is refused at Seed-Child.**
  The refusal names the profile, the recipe and the recipes the profile covers, and routes to Stuck as an unknown profile does.
  The board upsert keeps checking only the profile's name, since the board resolves no recipe.
- **An old-shape `profiles.yaml` fails to load with a way forward.**
  The refusal names the profile, the recipe → module → keys shape and `lyx config profiles`.
  The seed-only reconcile never rewrites a present file, so no migration is added; the hub's file is rewritten to the ladder by hand.

## Constraints

- `PATTERN-config-layers`: `runprofile` keeps its own stance of stacked layers with no template fill.
- `PATTERN-tolerant-records`: unchanged; profiles are config, not a Go-written record.
- `PATTERN-refusal-way-forward`: every new refusal names its way forward.
- The run settings validation stays the one `configreg.PrepareRunSettings` step.

## Auto-mode assumptions

- Recipe block keys are the board's recipe names, `loom` and `darn`; the loader checks only their shape, and Seed-Child's lookup refuses a recipe the profile lacks.
- `lyx board profiles` gains a top-level `defaults` field in its JSON envelope.

## Open risks

- Between the binary deploy and the rewrite of the hub's `profiles.yaml`, `lyx board profiles`, a board write naming a profile and Seed-Child refuse the old file.

## Acceptance criteria

- Loading the template yields `docs`, `light`, `solid`, `speed` and `wide`, and defaults `loom: solid` and `darn: light`.
- A profile, recipe block or module block of the wrong shape, a profile named `defaults`, and a default naming an unknown profile or one without that recipe's block, are refused naming the offending entry.
- Seed-Child writes only the recipe's block: the same profile yields `loom.code_max_bounces` 1 for a loom task and the darn column's value for a darn task.
- A task with no profile takes its recipe's default profile, and a recipe without a default writes only the task's own settings.
- A named profile without the task's recipe's block routes Seed-Child to Stuck with a way forward.
- `lyx board profiles` prints the defaults.
- The configreg profiles check refuses a recipe block holding an unknown key, naming the profile and the recipe.
