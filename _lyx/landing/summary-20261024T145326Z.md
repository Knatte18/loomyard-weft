# Run profiles as one ladder: speed, light, solid, wide

The hub's run profiles become one ladder that darn and loom share by name.
Each profile holds one settings block per recipe, and each recipe names a default profile.

## What changes

- **Profile shape.** `profiles.yaml` now maps profile → recipe (`loom`, `darn`) → module → keys.
  Seed-Child writes only the block of the recipe the task runs, then lays the task's `run_settings` over it.
  Darn's Code-Review perch reads loom's `code_*` keys, so the same key now takes the loom value in a loom run and the darn value in a darn run.
- **Defaults.** A reserved top-level `defaults` entry maps a recipe to its default profile.
  A task whose `profile` is empty takes that default; a recipe without one runs on the hub's config, as before.
  `Load` refuses a default that names an unknown profile, or a profile with no block for that recipe.
- **Template.** The seed is the ladder below, with every perch's hard cap set to its rounds plus one.
  It also holds `docs`, a darn-only profile with `darn.writer: sonnet[medium]` and one Code round, and `defaults: {loom: solid, darn: light}`.

  | Profile | Discussion | Plan | Code (loom) | Code (darn) |
  |---|---|---|---|---|
  | speed | 0 | 0 | 1 | 0 |
  | light | 1 | 1 | 1 | 1 |
  | solid | 2 | 2 | 1 | 2 |
  | wide | 3 | 2 | 1 | 3 |

- **Refusals.** A profile with no block for the task's recipe sends Seed-Child to Stuck.
  The message names the profile, the recipes the profile covers, and the way forward.
  The configreg profiles check now names the profile and the recipe of a refused key.
  An old-shape file fails to load, naming the recipe → module → keys shape and `lyx config profiles`.
- **`lyx board profiles`** prints a `defaults` field beside the profiles.
- The docs, the `ly:board` skill and the orch role stencil describe the new shape, the ladder and the defaults.

## Design decisions

- **A round count of 0 needs no entry gate.** The `<perch>_max_bounces` keys already settle a perch's Bouncer approved with no review round at 0, so `speed` turns perches off through profile settings alone.
- **The defaults live in `profiles.yaml`, not in a recipe module key.** A key in a recipe module could also be set through `run_settings`, where it would mean nothing.
- **Recipe coverage is checked only at Seed-Child.** The board upsert still checks only that a named profile exists, because the board does not resolve a task's recipe.
- **A speed perch with 0 rounds gets a hard cap of 1.** A hard cap of 0 means "take the review default", so rounds plus one keeps the rule uniform.
- **The old shape is not migrated.** The seed-only reconcile never rewrites a present file, and the old profiles do not map onto the ladder.

## After landing

After the deploy, the hub's own `profiles.yaml` on Bolt is still in the old shape and is refused until it is rewritten.
Until then, `lyx board profiles`, board writes that name a profile, and Seed-Child all fail.
To fix it, replace the file from the hub's prime worktree with the new `internal/runprofile/template.yaml`.
