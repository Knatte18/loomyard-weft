# Run profiles as one ladder: speed, light, solid, wide

The hub's run profiles (`budget`, `light`, `docs`, `darn-wide`) grew one at a time and their names said little.
They are replaced by one ladder, `speed`, `light`, `solid` and `wide`, that darn and loom share by name, plus `docs` as a darn-only profile.
Each rung sets the review rounds per perch, and each perch's hard cap is its rounds plus one, so `lyx loom circling continue` can grant one more round.

| Profile | Loom Discussion | Loom Plan | Loom Code | Darn Code |
|---|---|---|---|---|
| speed | 0 | 0 | 1 | 0 |
| light | 1 | 1 | 1 | 1 |
| solid | 2 | 2 | 1 | 2 |
| wide | 3 | 2 | 1 | 3 |

`docs` sets `darn.writer: sonnet[medium]` and one Code perch round, for pure prose and config value edits.

## What changes

- `profiles.yaml` takes a new shape: profile name → recipe (`loom` or `darn`) → module → keys.
  Each recipe block has the shape of a board task's `run_settings`.
  A reserved top-level `defaults` entry maps a recipe to its default profile; the template seeds `loom: solid` and `darn: light`.
- Seed-Child (`internal/battencli`, `internal/battenshed`) resolves the task's own profile, else its recipe's default, and lays the task's `run_settings` over that recipe's block only.
  A recipe without a default runs on the hub's config as before.
- `internal/configreg` validates each recipe block of each profile as run settings, and a refusal names the profile and the recipe.
- `lyx board profiles` prints a top-level `defaults` field beside the profiles.
  An empty `profile` on a board entry now means "take the recipe's default" instead of "no profile".
- Docs follow in the `doc.go` files of `runprofile`, `battenshed`, `boardcli`, `boardengine` and `configcli`, in `docs/overview.md`, and in the board skill and orch template role text that mention profiles.

## Design decisions

- **One block per recipe.**
  Darn's Code-Review perch reads loom's `code_*` keys, so the Code and Darn ladder columns are one key.
  A per-recipe block lets one profile name carry Code 1 for loom and 0–3 for darn without per-recipe key names.
- **A round count of 0 switches a perch off through profile settings alone.**
  The `<perch>_max_bounces` keys already settle a perch approved with no review round at 0, so no entry gate was added.
- **The defaults live in `profiles.yaml`**, not in a recipe module key, which `run_settings` could also set to no effect.
  A profile cannot be named `defaults`.
  A default naming no profile, or a profile with no block for that recipe, is refused at load.
- **A named profile with no block for the task's recipe is refused at Seed-Child**, naming the profile, the recipe and the recipes the profile covers; the task routes to Stuck.
  The board upsert still checks only that the named profile exists, since the board resolves no recipe.
- **An old-shape `profiles.yaml` fails to load** with a refusal that names the new shape and `lyx config profiles`.
  The seed-only reconcile never rewrites a present file, so there is no migration.

## To do by hand

- Rewrite the hub's own `profiles.yaml` on Bolt to the ladder from the hub's prime worktree after the binary deploys; this change does not touch it.
  Until then `lyx board profiles`, a board write naming a profile, and Seed-Child refuse the old file.
- Not run: the full gate beyond the packages' own tests; `lyx gate test --upto 2` on `main` before pushing is the check.
