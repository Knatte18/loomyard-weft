# config: repo-wide default plus per-worktree override

`config-overlay` · Note · feature, config

From `manifest/roadmap.md` Someday.
Every module's config resolves only from `<cwd>/_lyx/config/<module>.yaml` today, per-worktree with no shared default (`fabric.yaml` is the sole exception, anchored at `_board`).
Add a repo-wide default layer read from `_board`, with each worktree's own file as an override on top: the two-layer overlay millhouse already uses with `config.local.yaml`.
Not yet designed.

## Local override layer (operator, 2026-10-11)

A further layer on top: an untracked local config file under `.lyx/` per module, holding only the keys it overrides, as Millhouse's `config.local.yaml` did.
Batten would write a task's run settings there (`task-run-settings`), so they never show up as a tracked change to `_lyx/config` and landing does not flag them.
