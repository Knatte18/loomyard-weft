# A window whose home desktop no longer exists keeps that home after it is placed elsewhere

`kwin-home-desktop-gone` · Note · bug, termwindow

Depends on: [`kde-window-placement-build`](design-kde-window-placement-build.md)

## Seen (2026-11-28, Webster friction note of `kde-window-placement-build`)

A batch fork reading `internal/kwinplace/places.go` noted that a window whose home desktop no longer exists is placed on another desktop but keeps the old home.
It left this unchanged as outside its cards, and nobody has verified it.

## Idea

Verify the case; when it holds, the placement records the desktop it used as the window's new home, or states why the old home is kept.
