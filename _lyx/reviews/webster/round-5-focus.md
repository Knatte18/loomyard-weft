---
round: 5
focus:
  - "In internal/reedengine/windowsize.go, `pinGeometryOptionsLocked` installs the unseen-clear hook with `set-hook`: does every tmux call that takes a pane or window target there, and its expected argv in faketmux_test.go, use a target form real tmux accepts, and does selecting an own window clear `@lyx_unseen` against a real tmux?"
  - "In internal/reedengine/window.go, `OpenWindow` creates a window and chains `set-option` calls for `remain-on-exit` and the hidden mark: when an own window with the same requested name already exists, do the options land only on the new window, does the existing own window stay unchanged, and does the read-back by window id succeed?"
  - "In internal/reedengine/ownwindow_integration_test.go, the `InvalidConfigStillRemovesAndReportsTheKey` subtest asserts a pane is gone: does every assertion that treats a tmux error as proof of a missing pane use a check that reports a missing pane under tmux 3.6, such as membership in the `list-panes -s` listing?"
---

Round 4's three BLOCKING findings were fixed in the latest commits on the branch; these entries ask the next reviewer to confirm the fixes hold against a real tmux, not only the fake.
