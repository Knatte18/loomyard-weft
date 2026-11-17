---
round: 2
focus:
  - "internal/beamcli/hubstop.go spawnDetachedStop: settle which directory the detached `lym beam stop --all --detached` child starts in, and whether that directory satisfies lymcwd.Resolve's exact-anchor requirement in a hub whose prime has a subpath anchor."
  - "internal/beamcli/hubstop.go hubStopWorktrees and runHubStop: settle whether a worktree whose _lym junction or origin cannot be read is still put in the task-pair list, and whether pause_lifted reads true after a ConfirmationLifted outcome."
  - "internal/orchcli/stop.go beamDown, its BeamDown doc and internal/orchcli/doc.go: settle whether the docs match what the strand removal does to a caller inside the orch's pane, and whether a failed dobby read or end still ends the orch's registered process."
  - "internal/beamcli/hubstop_test.go, hubstop_integration_test.go, start_integration_test.go and internal/orchcli/beamdown_test.go: settle whether each stop --all path (slug-beside-all refusal, spawn arguments, worktree split, each runHubStop step failure, endOrch failure) is pinned without duplicating an existing test's coverage."
---

Focus targets the code the round-1 fixes touch.
