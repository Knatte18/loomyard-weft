# Reed key bindings stay stale on a running tmux server after a deploy

`reed-bindings-repin` · Note · bug, reed

Imported from [issue #646](https://github.com/Knatte18/loomyard/issues/646).

Seen 2026-10-15 after viewer-off-shell landed (474fb6680, the viewer follows the run until the operator moves it) and the Land row deployed it: Alt+Up/Down in the reed sessions of sibling-overlap-check and plan-test-file-path no longer switched windows for good. The tmux server still held the old root-table M-Up/M-Down bindings (no set-option @lyx_follow off), because pinBindingsLocked (internal/reedengine/bindings.go) runs only at boot and in the attach pre-flight. @lyx_follow was unset, which follow.go reads as on, so newAgentWindowTakesViewerLocked (ownwindow.go) pulled the viewer back each time a run opened a new agent window. Workaround: run lyx reed attach once in any pair to re-pin the bindings server-wide, or tmux set-option -t <session> @lyx_follow off. Expected: a new lyx binary re-pins the bindings on a running server without waiting for an attach (or an unset follow option reads as off).