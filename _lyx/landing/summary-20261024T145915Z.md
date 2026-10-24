# Run profiles as one ladder: speed, light, solid, wide

The hub's run profiles (`budget`, `light`, `docs`, `darn-wide`) grew one at a time and their names said little.
They are replaced by one ladder, `speed`, `light`, `solid` and `wide`, that darn and loom share by name, plus `docs` as a darn-only profile.
Each rung sets the review rounds per perch, and each perch's hard cap is its rounds plus one, so `lyx loom circling continue` can grant one more round.

## What changes

- `profiles.yaml` changes shape to profile → recipe → module → keys.
  A profile holds one settings block per recipe, `loom` and `darn`, each with the shape of a board task's `run_settings`.
- A reserved top-level `defaults` entry maps a recipe to its default profile.
  The template seeds `defaults: {loom: solid, darn: light}`.
- Seed-Child resolves the task's own profile, else its recipe's default, and lays the task's `run_settings` over that recipe's block only.
  Darn's Code-Review perch reads loom's `code_*` keys, so the same profile gives `loom.code_max_bounces` 1 in a loom run and the Darn column's value in a darn run.
- `lyx board profiles` prints `defaults` beside the profiles.
- The `configreg` profiles check validates each recipe block of each profile as run settings, and names the profile and recipe of a bad key.
- Docs: `runprofile`, `battenshed`, `boardengine`, `boardcli` and `configcli` `doc.go`, `docs/overview.md`, the board skill and the orch template role.

## Ladder

| Profile | Loom Discussion | Loom Plan | Loom Code | Darn Code |
|---|---|---|---|---|
| speed | 0 | 0 | 1 | 0 |
| light | 1 | 1 | 1 | 1 |
| solid | 2 | 2 | 1 | 2 |
| wide | 3 | 2 | 1 | 3 |

`docs` has only a darn block: `darn.writer: sonnet[medium]` and one Code perch round.

## Design decisions

- A round count of 0 turns a perch off through profile settings alone, since the `<perch>_max_bounces` keys already settle a perch's Bouncer approved with no review round at 0.
  No entry gate is added.
- A profile cannot be named `defaults`.
  A default naming no profile, or a profile with no block for that recipe, is refused at load.
  A recipe without a default runs on the hub's config as before.
- A named profile with no block for the task's recipe is refused at Seed-Child, naming the profile, the recipe and the recipes the profile covers, and routes to Stuck.
  The board upsert still checks only that the profile exists, since the board resolves no recipe.
- An old-shape `profiles.yaml` fails to load with a refusal that names the new shape and `lyx config profiles`.
  The seed-only reconcile never rewrites a present file, so no migration is added.

## Check by hand

- Rewrite the hub's own `profiles.yaml` on Bolt to the ladder after the binary deploys.
  Until then `lyx board profiles`, a board write naming a profile and Seed-Child refuse the old file.
- Run `lyx board profiles` against the rewritten file and confirm `defaults` and all five profiles print.
