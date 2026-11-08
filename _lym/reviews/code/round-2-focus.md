---
round: 2
focus:
  - "internal/reedengine/ownwindow_integration_test.go, TestOwnWindow_RealTmux: is any subtest that works on one OS only (the resize-hook subtest needs tmux set-hook, which psmux lacks) left out before t.Run on the other OS, as the file already does near its line 146, rather than registered and returning early?"
  - "internal/reedengine/ownwindow.go evenLayoutResizeHook guards on #{?window_zoomed_flag,0,1}: does a test fail if that guard is removed, by zooming a pane of a shared window, resizing the window, and asserting the window is still zoomed?"
  - "internal/reedengine/ownwindow.go relayoutSharedWindowsLocked pins each window's resize hook after the layout, and the docs say it is pinned again after a strand leaves: does a test fail if that pin is removed, by removing a strand, resizing the remaining window, and asserting equal pane widths?"
---

Entries name sites in the committed diff `539525a02bc86d7d4ef3a6afdf95989cdc52080d..HEAD`.
Review the whole diff as usual; these are the questions to settle first.
