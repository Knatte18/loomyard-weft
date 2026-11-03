# Reconcile renames former recipe blocks in profiles, lists missing ones

A hub whose `profiles.yaml` was seeded before the `loom` → `tapestry` recipe rename kept `loom` recipe blocks and a `defaults.loom` entry.
No run reads those, so every tapestry run on such a profile stopped at Seed-Child.
`lym config reconcile --layer default --apply` changed nothing, because `profiles` is seed-only and reconcile never touched a present seed-only file.

## What changes

- **Rename on reconcile.**
  `runprofile.MigrateConfig` renames, in every profile and under `defaults`, a recipe block or entry keyed by a recipe's former name to its current name, keeping the operator's values and comments.
  If a profile already holds the current name, it keeps that block and drops the former one.
  Each rewrite is one line, such as `light.loom: renamed to light.tapestry`.
  The former names live in one table, `recipeRenames`, holding `loom` → `shedrun.RecipeTapestry`.
- **Missing blocks reported, never added.**
  `runprofile.LackingTemplateBlocks` lists each recipe block or `defaults` entry that a template profile holds and the hub's profile of the same name lacks, such as `light.sampler`.
  A profile the template lacks, or one the operator removed whole, lists nothing.
- **Seed-only files are migrated.**
  `configreg.Module` gains a `Lacking` hook, and the `profiles` row sets both `Migrate` and `Lacking`.
  `configsync.ReconcileHubWideAt` now runs a seed-only module's `Migrate` hook over a present file, or over the prime's copy that seeds an absent default.
  It writes the file only when the hook rewrote something.
  The `Lacking` report goes in the new `Result.Lacking` field.
  A seed-only file is still never filled or pruned.
- **Reporting.**
  `lym config reconcile --layer default` shows the rewrites under `migrated` and the report under `lacking`.
  The hub reconcile walk, which a start verb runs after a binary change, logs one Warn per module whose file lacks template units, with `lym config <module>` as the way forward.
- **Seed-Child way forward.**
  `Profiles.Settings` refuses a profile that holds the task's recipe only under its former name.
  `Profiles.Default` now returns an error too, and refuses a `defaults` entry held only under the former name, which would otherwise let the task run silently on the hub's own config.
  Both refusals name `lym config reconcile --layer default --apply` from the prime worktree as the way forward.
  Seed-Child routes them to Stuck through `ErrInvalidRunSettings`.
- Docs follow: the `runprofile`, `configcli`, `hubreconcile` and `configengine` package docs, the `configreg.Module` field docs, and the `profiles.yaml` template header no longer say a seed-only file is never rewritten.

## Design decisions

- **Report the missing `sampler` blocks rather than add them.**
  The file is operator-owned, and reconcile cannot tell a block the operator removed on purpose from one that was never seeded.
  Adding blocks from the template would undo a deliberate removal on every binary change.
- **A new `Lacking` hook instead of reusing `Migrate`.**
  `Migrate` lines mean a rewrite, and a non-empty set causes a write.
  A report that never changes the file needs its own channel, or the file would be rewritten on every pass.
- **The per-worktree path is unchanged.**
  `reconcileBaseWorktreeFile` (a standalone worktree with no board) and the sparse layers still leave a seed-only file untouched.
  `profiles` is hub-wide, and the bug lives at the board dir.
- The `!m.HubWide && presentSeedOnly` early return in `ReconcileHubWideAt` is gone, so a present per-worktree seed-only default (`models`) takes the same seed-only branch.
  `models` has no hooks, so its result is unchanged.

Closes #29.

## To check by hand

- On a copy of a pre-rename hub's `profiles.yaml` (`loom` blocks, `defaults.loom`, no `sampler`):
  - `lym config reconcile --layer default` lists `migrated` lines for each renamed block and `lacking` entries for each `<profile>.sampler`.
  - `--apply` rewrites the file with `tapestry` blocks holding the same values.
- A tapestry task naming a profile that still holds a `loom` block stops at Seed-Child with the reconcile command as its way forward.
