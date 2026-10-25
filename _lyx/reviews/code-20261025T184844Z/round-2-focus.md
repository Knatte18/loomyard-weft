---
round: 2
focus:
  - "internal/battencli/overview.go, atOrBelow and orchsAtOrBelow: does the walk up the parent chain terminate for every input, including a self-parent and two beams naming each other as parent with the window's worktree outside the cycle, and does the caller leave the cyclic orchs out without hanging the frame?"
  - "internal/battencli/overview.go orch comments and windowOrchPaths: does every comment about the order of the shown orchs match the order windowOrchPaths actually returns?"
---

Targets the code the round-1 fixes touch.
