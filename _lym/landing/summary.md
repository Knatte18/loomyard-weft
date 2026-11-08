# reedengine: keep a group window's panes equal after a resize

When two strands of one group share a window, such as a Claude reviewer and a Codex assist reviewer, reed lays the window out evenly.
After a resize of the terminal window, tmux's own layout resize could leave one pane at its old width while the other took the whole change.
The Codex pane then kept its width and the Claude pane shrank.

Reed now pins a `window-resized` hook on each shared group window.
The hook issues the same even layout again after every resize, so each pane keeps an equal share in the `window_split` direction.

## Design decisions

- The hook is set by `pinEvenLayoutOnResizeLocked` in `internal/reedengine/ownwindow.go`.
  It runs wherever reed lays out a shared window: after a pane joins or opens the window (`createOwnWindowPaneLocked`), and after a strand leaves it (`relayoutSharedWindowsLocked`).
- The hook replaces the window's earlier one, so each window carries a single entry naming the current layout.
  After a leave under a changed `window_split`, the hook follows the new direction.
- The hook skips a zoomed window (`window_zoomed_flag`), since a layout would unzoom it.
- psmux lacks `set-hook`, so on Windows no hook is pinned and a resize keeps whatever split tmux gives.
  `set-hook` stays out of `requiredSubcommands`, so the capability probe is unchanged.
  A failure to set the hook is logged and never fails the step.
- `windowResizedHookName` is shared with the shell window's unpin code.
  The shell window is unaffected, since reed still arranges no panes there.
- `codexengine`'s `--no-alt-screen` is untouched.
- `internal/reedengine/doc.go` records the new hook.

## Tests

`TestOwnWindow_RealTmux` gains subtests that run against real tmux and are left out on Windows:

- A resized shared window gets equal pane widths again.
- A resize leaves a zoomed shared window zoomed.
- The hook is pinned again after a strand leaves, so a resize keeps the `height` split.

## To check by hand

- `os-bound-subtest-registered-on-windows` (`internal/reedengine/ownwindow_integration_test.go`): confirm the resize subtests are not registered on Windows.
  They are guarded by `runtime.GOOS != "windows"`, but the Windows build was not run.
- `resize-hook-zoom-skip-untested` (`internal/reedengine/ownwindow_integration_test.go`): confirm `ResizeLeavesAZoomedSharedWindowZoomed` fails without the `window_zoomed_flag` check in `evenLayoutResizeHook`.
- Resize a real terminal with a Claude reviewer and a Codex assist reviewer side by side, and confirm both panes scale together.
