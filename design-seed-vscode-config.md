# Seed the VS Code config of each pair when it is made

`seed-vscode-config` · Note · enhancement, ide, batten

## Idea (operator, 2026-10-13)

A task pair should carry its VS Code config from the start, so opening it with plain `code .` gives what `lyx ide spawn <slug>` gives.
It costs little, and saves going through `lyx ide` when the operator wants a run's worktree in VS Code.

## Seen (2026-10-13)

`lyx ide spawn` (`internal/ideengine/spawn.go`) does the setup on every call, then launches VS Code:

- assigns the worktree a title-bar color;
- writes the worktree's `.vscode/` config, including the interactive task chain that runs on folder open, and keeps `.vscode/` out of git through `info/exclude`;
- seeds the lyx block of terminal key bindings into the user's VS Code `keybindings.json`.

Nothing writes this when a pair is made, so a pair never opened through `lyx ide` has no `.vscode/` and no color.

## Scope to discuss

- Where the seed runs: a batten step at the start of a run (beside Seed-Child), or `lyx fabric add` for every pair.
- Which parts: the color and `.vscode/` config per worktree; the user-level keybindings stay with `lyx ide`.
- Whether the folder-open task chain should be seeded too, since `code .` then runs it on open, or only the passive settings.
- `lyx ide spawn` keeps regenerating on every call, so a seeded pair still picks up a newer chain.
