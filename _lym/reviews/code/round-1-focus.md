---
round: 1
focus:
  - "internal/reedengine/ownwindow.go `pinEvenLayoutOnResizeLocked`: does the `set-hook -w -t <target> window-resized` option stay scoped to the group window, and does the hook's command run against the window that fired it?"
  - "Callers of `windowResizedHookName` outside the diff (the window-size code that unpins the shell window): can its set or unset touch a group window's hook, or the reverse, so the two hooks overwrite or remove each other?"
  - "internal/reedengine/spawn.go `createOwnWindowPaneLocked` and ownwindow.go `relayoutSharedWindowsLocked`: is the hook pinned on every path that lays a shared window out (open, join, leave), and skipped on the paths where layout is skipped, such as an invalid `window_split`?"
  - "`evenLayoutResizeHook`: is the tmux quoting of the `if-shell -F` command correct for a layout name, and is the zoom check right for the Codex assistant case in the Goal?"
  - "Does the diff make the change the Goal and the 2026-11-08 addition ask for, panes sharing width evenly on window resize, without touching codexengine's `--no-alt-screen`?"
  - "ownwindow_integration_test.go `ResizeLaysTheSharedWindowOutEvenlyAgain`: does it fail without the hook, and does it duplicate the coverage of an existing sibling subtest?"
  - "doc.go and the commit message and `_lym/landing/summary.md` if present: does each claim (Windows pins none, zoomed window skipped, hook replaced per layout) match the code?"
---

Seed focus for round 1.
The diff is small and confined to `internal/reedengine`.
The main risks are hook interaction with the existing window-size unpin and the quoting of the hook command.
