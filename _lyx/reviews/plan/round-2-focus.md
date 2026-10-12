---
round: 2
focus:
  - "Card 7 (`_lyx/plan/07-reed-own-window-bar.md`) pins a `session-window-changed` set-hook in `pinGeometryOptionsLocked`, which `AttachArgv` (`internal/reedengine/attach.go`) calls before its suppressing guards. Settle whether every existing test in `internal/reedengine/attach_test.go` that asserts on `set-hook` calls (for example `TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` and `TestAttachArgv_PreflightOnAKnownGoodSession`) is listed in a card's `**Edit:**` whose change breaks it."
  - "Card 1 (`_lyx/plan/01-render-own-window-display.md`) makes `render.ValidateDisplay` accept own-window displays. Settle whether any existing test row in `internal/reedengine/attach_test.go` that relied on own-window being rejected still pins what its name says between card 1 and card 3, and whether a card targets it."
  - "Card 4 (`_lyx/plan/04-reed-own-window-launch.md`) kills a relaunching strand's dead own-window pane, and card 5 (`_lyx/plan/05-reed-window-groups.md`) then places the strand in a shared window. Settle whether the plan keeps this consistent with the decision record's rules on when reed may kill an own-window pane and on selecting the viewer's window, and whether card 5 places from a window listing taken after the kill."
---

Focus targets the cross-card test-breakage and relaunch-ordering questions only; other plan areas are unrestricted as usual.
