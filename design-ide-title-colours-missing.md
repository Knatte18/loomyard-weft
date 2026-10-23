# Seeded VS Code title-bar colours do not show in task worktrees

`ide-title-colours-missing` · Note · bug, ide

## Seen (orch, 2026-10-23)

`vscode.WriteConfig` (`internal/vscode/config.go`) writes `workbench.colorCustomizations` with a title-bar colour picked from the palette in `internal/vscode/color.go`, and writes `settings.json` only when the file is absent.
In the `loom-tail-review` worktree, seeded about 11:15, `.vscode/settings.json` holds every other seeded key but no `workbench.colorCustomizations`, and its modification time is 13:27, after seeding; no other lyx code writes that file.
The Peacock extension (`johnpapa.vscode-peacock`), which manages `workbench.colorCustomizations`, is installed on OSL-1046.
Since `85c09cee` (2026-10-19) the operator opens a hub through one multi-root workspace file listing the prime, Bolt and every worktree.

## To check

- Whether Peacock removed the key.
- Whether VS Code applies a folder's `workbench.colorCustomizations` at all in a multi-root workspace window, where window-scoped settings come from the `.code-workspace` file; if not, the colour has had no effect since `85c09cee` whenever a worktree is opened through the hub workspace.

## Operator view (2026-10-23)

Every worktree's VS Code window gets a colour picked from a palette, as Millhouse does.
