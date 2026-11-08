# Reed keeps a group window's panes even through a resize

A window group's shared window, such as a Code-Review round's chair beside its Codex assistant reviewer, was laid out evenly only when a pane joined or left it.
A later terminal resize was left to tmux's own resize of the layout.
On Linux that shrank the Claude chair's pane while the Codex pane kept its old width, so the two panes no longer shared the window evenly.

Every even layout of a shared window now also pins the window's `window-resized` hook to issue the same layout again.
tmux runs the hook after its own resize, against the window that fired it, so the window's panes return to equal shares on every grow or shrink.
That covers both the layout after a pane joins or opens the window (`createOwnWindowPaneLocked`) and the layout after a strand leaves it (`relayoutSharedWindowsLocked`).
The hook is `if-shell -F '#{?window_zoomed_flag,0,1}' 'select-layout <layout>'`, so a zoomed window stays zoomed.
Dragging the border between the panes resizes no window, so a split the operator sets by hand stays until the next resize.

## Design decisions

- The fix is in reed's layout handling, not in Codex's launch: the operator saw the pane border itself stay put, and `codexengine` keeps `--no-alt-screen`, as the decision record's added entry rules.
- The hook is window-scoped and takes no `-t` in its command, so it lays out only the window that fired it and never another window.
  It replaces the window's earlier entry, so each layout leaves one entry naming the current `window_split` direction.
- The shell window's `window-resized` unpin is unchanged: it clears the shell window's own hook, and a group window's hook is a separate window option.
- psmux lacks `set-hook`, so on Windows no hook is pinned and a resize keeps whatever split tmux's own resize gives, as before.
  A failed pin is logged at Warn and never fails the launch or the removal.

## Tests

`TestOwnWindow_RealTmux` gains `ResizeLaysTheSharedWindowOutEvenlyAgain`.
It opens a group of two, narrows one pane, grows the window and waits until both pane widths are within one column of each other.
