# Motif runs: own profile block, a discussed status, and live board status

Fixes three motif faults seen in hub Models-LYMHUB (#58, #53, #49).
A motif task now takes its own run-profile block.
A finished motif shows as `discussed` on the board instead of `draft`.
The board status follows batten's rows and then the inner run's first row while that row runs.

## What changes

- **Motif's run profile (#58).**
  `motif-recipe.yaml` no longer sets `profile_recipe: tapestry`.
  A motif task takes the `motif` block of its profile and the `motif` default, like every other recipe.
  The `profiles.yaml` template gains a `motif` block in `speed`, `light`, `solid` and `wide`, holding the Discussion perch values of the profile's tapestry block, and `defaults.motif: solid`.
  No recipe sets `profile_recipe` any more, so the key is removed from the recipe format (`shedbuild.Recipe.ProfileRecipe`, `loomrecipe.ProfileRecipe`, the shed recipe spec), and Seed-Child's `prepareRunSettings` reads the task's recipe name directly.
- **A finished motif reads as discussed (#53).**
  A finished run of a recipe with no Finalize row now writes the new status `discussed` (`boardengine.StatusDiscussed`) instead of `draft`.
  Like `draft`, it is not a run status, so the run lock is released and dobby does not start the entry again; a null status releases it.
  `ComputeLayers` puts such a task in a new `Discussed` layer.
  The README renders it under a Discussed subsection between Independent and Draft, its title led by a `discussed` code span.
  `lym board list` reports layer `Discussed`, and the `--text` listing shows `[discussed]`.
- **The board status follows the run (#49).**
  - `shedengine`: a step over a status file that names no row now writes the first row as `current_producer` and tells `CommitStatus` before calling that row, alongside the existing resume write.
    A run whose first row is a long writer, as motif's Discussion-Write is, now shows that row in `lym loom status` and on its board entry.
    Before, both showed no producer until the row's verdict.
  - `battencli`'s status seam writes `<state> · <row>` on the task's entry at each batten row before Run-Shed, from the Worktree-Create claim until the inner run takes over.
    `battenrecipe.RowsBeforeRunShed` derives those rows from the recipe.
    A failed board write only warns, as loom's does.
  - `Board.SetRunStatus` writes nothing when the entry already carries the status, so a repeated transition makes no Bolt commit.

## Design decisions

- `discussed` is a new status value rather than the `discussed` flag read beside `draft`.
  This way the JSON listing, the `--text` listing and the README all show it from the status alone.
  A tapestry run that reused a stored record and was then cancelled still reads as `draft`.
- Batten writes the board status only for rows before Run-Shed.
  From Run-Shed on, the inner run owns the entry's status.
  A batten write at Run-Shed, Land or Worktree-Teardown would overwrite the inner run's producer or a motif's `discussed`.
- The motif default is `solid`, the tapestry default, so a motif run on a freshly seeded hub reviews as many Discussion rounds as before.

## To check by hand

- `profiles.yaml` is seed-only, so an existing hub keeps a file with no motif blocks and no motif default.
  On such a hub, a motif task naming no profile runs on the hub's own `loom.yaml`.
  One naming a profile is refused at Seed-Child: the profile holds no motif block, and the refusal names adding one with `lym config profiles`.
  `lym config reconcile` lists each missing motif block under `lacking`.
  Models-LYMHUB needs those blocks and `defaults.motif` added by hand.
- Motif entries already left at `draft` by an earlier run stay `draft`; only new motif runs write `discussed`.
- The start write adds one status commit at the start of every shed run, batten's own included.
