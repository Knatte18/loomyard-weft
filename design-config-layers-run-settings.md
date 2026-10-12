# Config layers and per-task run settings

`config-layers-run-settings` · Task · feature, config, batten, board · running · Webster-Bouncer

Merged from notes `config-overlay` and `task-run-settings` (operator, 2026-10-11).

## Seen

- Every module's config resolves only from `<cwd>/_lyx/config/<module>.yaml`, per worktree with no shared default (`fabric.yaml` is the sole exception, anchored at `_board`); the embedded template is used only when the file is absent.
  A present file that lacks a key gets the Go zero value, not the template's default: after `c279a38ba` added `webster.yaml` `kickstart_pack` (template default `true`), a pair created before `lyx config reconcile --apply` ran with the pack off (2026-10-11).
- The orch sets `review_max_bounces: 2` by hand in a small task's pair `loom.yaml` right after batten creates the pair, and must first check that the pair exists, since a run can stall at Worktree-Create.
  Landing then warns that the task branch changed a per-worktree config file, which needs nothing but costs a look each time.
- The board entry already carries one run setting, `recipe`.

## Scope

### 1. Config layers

From `manifest/roadmap.md` Someday: a repo-wide default layer read from `_board`, with each worktree's own file as an override on top, the two-layer overlay Millhouse used with `config.local.yaml`.

On top of that, a local override layer (operator, 2026-10-11): an untracked file under `.lyx/` per module, holding only the keys it overrides, as Millhouse's `config.local.yaml` did.

### 2. Run settings on the board task

Run settings are part of the task (operator, 2026-10-11): the board entry carries them, and batten applies them to the pair when it creates it, by writing the pair's local override file, so they never show up as a tracked change to `_lyx/config` and landing does not flag them.
A task may carry any key of any module's config, and overrides only the keys it names (operator).
Typical use: review rounds per perch (Discussion, Plan and Webster review segments) and darn's review budget for a small task.
Starting a ready task then needs no hand step, which `orch-split` needs for a mechanical run driver.

## Open

- Whether a key missing from a present file falls back to the layer below and finally to the template's default, which would also close the `kickstart_pack` gap above.
- How `lyx config reconcile` and the `lyx config` editor show which layer a value comes from.
