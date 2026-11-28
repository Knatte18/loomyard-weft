# A lym verb puts the hub windows back in their desktops and tiling zones when they have drifted

`window-retile-verb` · Task · feature, termwindow, desktop

Depends on: [`kde-window-placement-build`](design-kde-window-placement-build.md), [`kwin-home-desktop-gone`](design-kwin-home-desktop-gone.md)

## Seen

The KDE placement task places a lym window only when the window opens, and lym moves no window by itself afterwards.
A window the operator drags out of its zone, or one KWin moves on its own (seen with two screens, 2026-11-27), stays out of place until it is opened again.
The operator asked (2026-11-28) how windows that have come out of sync get tiled again.

## Idea

A verb, run by hand, that re-applies the stored placement to the hub's open lym windows:
each goes back to its home desktop and zone, tiled, through the same KWin script path and lock the opening placement uses.
It moves only lym windows of the hub, and reports what it moved and what it could not place.

## Open

- Name and home of the verb (for example under `lym window`), and whether it covers one worktree or the whole hub.
- Whether a desktop-bar button runs it.
