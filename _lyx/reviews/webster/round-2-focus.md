---
round: 2
focus:
  - "internal/reedengine/spawn.go `launchStrandLocked`: when an own-window strand whose pane is dead is relaunched through Resume or add --if-absent, is the strand's previous pane killed once the new pane is bound, including when that pane sits in a shared group window holding other strands? Settle that the window is never ended mid-launch and that the group window ends with one pane per strand."
  - "internal/reedengine/ownwindow.go placement constants (`placeApart`, `placeNewShared`, `placeJoinShared`): do they read as one iota sequence with no behavior change to placement decisions?"
  - "internal/reedengine `TestOwnWindow_RealTmux`: does a case cover a shared-group member dying and being relaunched, and does it pin behavior no existing test covers?"
---

Focus targets the code touched by the round-1 fixes, which no fresh reviewer has seen.
