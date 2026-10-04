# config: repo-wide default plus per-worktree override

`config-overlay` · Note · enhancement, config

From `manifest/roadmap.md` Someday.
Every module's config resolves only from `<cwd>/_lyx/config/<module>.yaml` today, per-worktree with no shared default (`fabric.yaml` is the sole exception, anchored at `_board`).
Add a repo-wide default layer read from `_board`, with each worktree's own file as an override on top: the two-layer overlay millhouse already uses with `config.local.yaml`.
Not yet designed.