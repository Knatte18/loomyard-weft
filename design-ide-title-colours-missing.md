# Seeded VS Code title-bar colours do not show in task worktrees

`ide-title-colours-missing` · Note · bug, ide

## Seen (orch, 2026-10-23)

`vscode.WriteConfig` (`internal/vscode/config.go`) writes `workbench.colorCustomizations` with a title-bar colour picked from the palette in `internal/vscode/color.go`, and writes `settings.json` only when the file is absent.
In the `loom-tail-review` worktree, `.vscode/settings.json` holds every other seeded key but no `workbench.colorCustomizations`; no other lyx code writes that file.
The operator opened VS Code on that folder: the title bar showed the seeded purple for a moment, then lost its colour, and the file's modification time is that opening.
The Peacock extension (`johnpapa.vscode-peacock` 4.5.1) is installed on OSL-1046.
It manages `workbench.colorCustomizations` from its own `peacock.color` setting, and its `applyColor` calls `unapplyColors`, which strips the title-bar colours, when the colour it is given is not a valid colour.
The seeder writes no `peacock.color`.

## Likely cause, not verified

Peacock starts with the window, finds no `peacock.color`, and strips the colour the seeder wrote.
Check: add `"peacock.color"` with the seeded colour to the folder's `settings.json`, or disable Peacock for that workspace, and reopen; the colour then stays if Peacock is the cause.

## Idea

The seeder also writes `"peacock.color": <the palette colour>`, so Peacock keeps and applies the same colour.
Also check whether a folder's colour applies in the hub's multi-root workspace window (`85c09cee`), where window-scoped settings come from the `.code-workspace` file.

## Operator view (2026-10-23)

Every worktree's VS Code window gets a colour picked from a palette, as Millhouse does.
