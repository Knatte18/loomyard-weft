# VS Code colours by project and worktree, and lym ide seed with no slug

`ide-seed-current-worktree` · Task · enhancement, ide, workspaces · running · Code-Burler-Review

## Seen

`lym ide seed <slug>` requires a slug, and its help names only task worktrees.
A fresh `lym fabric clone` leaves the prime with no `.vscode/`; seeding it took `lym ide seed loomyard` with the prime's name spelled out.
Every worktree today gets one title-bar colour from `vscode.PickColor`, the prime included (issue #882), and nothing shows which beam a worktree belongs to.

## Design (agreed with the operator, 2026-10-27)

Only two areas are coloured, since they are the only ones that keep their colour in an unfocused window (tested in VS Code 1.142 on Linux, custom title bar):

- **Title bar** (`titleBar.activeBackground` and `titleBar.inactiveBackground`, the same colour for both): the project's colour. The window manager draws the window edge in the same colour.
- **Activity bar** (`activityBar.background`): the worktree's own colour.

No other colour key is written: the status bar loses its background in an unfocused window while its text colour stays, `window.activeBorder` is not drawn, and tab colours change with the selected tab.

The colours:

- **Prime**: green title bar and green activity bar.
  The palette's green, `mainColor` and `palette[0]` in `internal/vscode/color.go`, becomes the darker `#1b5e20` in place of `#2d7d46`, which the operator found too light.
- **Task under the prime**: green title bar; activity bar any palette colour but green.
- **Beam**: picked when `lym beam create` makes the beam, from the palette less green and the colours other beams hold, and recorded on the beam so it stays the same at every open; title bar and activity bar both take it.
- **Task under a beam**: the beam's colour on the title bar; activity bar any palette colour but the beam's.
- Within one parent, a task's activity-bar colour avoids its sibling tasks' colours while the palette has one free.

Text colour follows the background's luminance: white on a dark colour, black on a light one.
The palette stays mostly dark, so text is mostly white.

## lym ide seed with no slug

The slug becomes optional: with none, `lym ide seed` seeds the worktree the current directory sits in, the prime included.
The help names the prime as well as task worktrees.
`fabric clone` stays IDE-free.

## Window title

Every worktree's `window.title` is the hub shortname upper-cased, a colon and the worktree's slug, such as `LY: loomyard` for the prime and `LY: <slug>` for a task.
A beam takes the same form with its own slug; a task under a beam names only its own slug, not the beam's.
The `.code-workspace` file `lym ide spawn` writes carries the same title and colours, since its settings override the folder's.
