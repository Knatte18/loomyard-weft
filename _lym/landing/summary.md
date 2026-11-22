# Read per-task beam settings by one lookup from the worktree up

## What changes

`pull_request` and `land_notice` each wired a beam value and a task value their own way, and the coming `autonomy` setting would have added a third rule.
They now share one lookup in `dobbyengine`, read at call time:

1. The value the task's own worktree sets applies.
   A task sets it through `run_settings`, which batten copies into its worktree like every other setting.
2. A worktree that does not set it takes its parent worktree's value, up through the beams to the prime.
3. When no worktree in the chain sets it, the hub's default layer applies, else the template's default.

"Sets" means written in that worktree's own files, read through the new `configengine.TopLevelValue`, never the merged value.
A hub default is therefore never a worktree's own setting.
A value set lower down overrides the one above in both directions.
`hubgeom.TaskSettingChain` tells the lookup the worktree chain.

User-visible changes:

- The board fields `pull_request` and `notify_on_land` are gone from `boardengine`.
  A task's value lives in `run_settings`.
- `pull_request` keeps `on`, `off`, `merge` and `merge-in`.
  The modes that only decided how far the board field counted, `required` and the field clause of `on` and `off`, are gone.
- `land_notice` takes `on` and `off`, with `off` as the template default.
  The `all` and `per-task` modes are gone.
- `landingshed` (pull-request mode, publish, PR review, finalize) and `loomcli` landing deps read the landing run's setting through the lookup.
- `pattern/PATTERN-config-layers.md` gains a "Per-task beam settings" section stating the rule, and `PATTERN-landing.md` is updated to match.

## Migration

A reinterpretation, `beam-per-task-settings`, ships through `hubreconcile`:

- It rewrites the hub's `beam.yaml`: `required` becomes `on`, `all` becomes `on`, and `per-task` becomes `off`.
- It flags a worktree's own `beam.yaml` that still holds a retired value, with the way forward, since only the hub file is rewritten.
- A `required` rewritten to `on` is flagged too, because `on` does not keep what `required` meant:
  `lym beam ready` no longer refuses a beam with no pull request recorded, and a task missing an origin fact lands locally.
- It carries the `pull_request` and `notify_on_land` values that existing board entries hold into `run_settings` of the live worktrees that need them.

## Design decisions

- One reader for every per-task beam setting, so `autonomy` in `orch-autonomy-mode` builds on it and this task lands first.
- The lookup reads file layers directly, with no template fill, and reads the template only when no file sets the key.
  That is a third strictness stance beside strict and degrading loads, recorded in `PATTERN-config-layers.md`.

## Check by hand

Findings a review left open, to check in the diff:

- `beam-project-notify-on-land-dropped`: a beam project's `notify_on_land` is dropped, and the migration may not carry it.
- `carry-misses-runs-in-flight`: the one-time carry of board fields into `run_settings` may miss a run already in flight.
- `duplicate-row-in-prepare-run-settings`: a duplicate row in the run-settings preparation.
- `member-line-overrides-explicit-off`: a member line may override an explicit `off`.
- `per-task-keys-declared-at-several-sites`: the per-task keys are declared at more than one site.
- `registration-test-lacks-keep-marker`: the registration test lacks the keep marker.
- `reinterpret-discovers-worktrees-and-anchors`: the reinterpretation discovers worktrees and anchors itself rather than being told them.

No regression check ran for this task, and no integration suite result is recorded.

## Merge danger

**Door:** one-way
**Blast radius:** dobbyengine
