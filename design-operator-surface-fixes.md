# Deploy and VS Code leave every worktree runnable and clean

`operator-surface-fixes` · Planned · bug

## Goal

Deploying lyx and opening worktrees in VS Code leave every worktree runnable and clean, with no manual step after either.

## Bugs

- #339: a new required config key shipped by a deploy makes loom refuse in every worktree until `lyx config reconcile --apply` runs there.
- #340: opening the prime in VS Code while `lyx orch` hosts the hub collapses the orch pane to 3 rows and stalls every cycle.
- #334: `lyx ide spawn` writes a lyx-managed block into a task pair's tracked `.gitignore`.

## Fix direction

- #339: a key that has a default is not required: loom fills a missing key from its default and logs it instead of refusing.
- #340: reed sizes its sessions to the latest client (`window-size latest`), and the folder-open task skips `reed add` when the worktree already has an orch strand.
- #334: `lyx ide spawn` records its ignore in `.git/info/exclude`, never in a tracked `.gitignore`.
  Rename the repo `.gitignore`'s "mill-managed" block, now that mill is gone.

## Done when

- A deploy that adds a defaulted key leaves running pairs runnable with no reconcile.
- Opening the prime in VS Code leaves the orch pane's size and session alone.
- `lyx ide spawn` on a pair leaves `git status` clean.
- CLAUDE.md's TEMPORARY items for #339, #340 and #334 are deleted, and the issues are closed by the landing commit.
