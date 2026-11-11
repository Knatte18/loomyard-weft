---
round: 1
focus:
  - "internal/shuttleengine/codexengine/sandbox.go `runSandboxProbe`: does `codex sandbox -c sandbox_mode=... -- true` match the Codex CLI's real subcommand and flag shape, and does the probe exercise the same policy, home and cwd the launch uses?"
  - "`launchSandboxMode` in Prepare and OpenLaunch: is the probe's `c.dirs.Anchor` a valid cwd on every path, including `New()` with empty dirs, and does a probe failure on a transient cause (timeout, missing binary) wrongly degrade auto to danger-full-access?"
  - "Goal item 2 (a seat that cannot start is reported with its cause, not ended as stopped): does the explicit workspace-write refusal in Prepare/OpenLaunch actually surface to the seat's reported state, and does the auto path's warning reach the operator?"
  - "Goal item 1 (no manual setting needed): does the template default `auto` plus `LoadConfig` validation and the machine-layer rule in `ConfigSpec` hold together, and do existing machine overrides pinning `workspace-write` or `danger-full-access` still behave as before?"
  - "Test coverage of the new behavior in sandbox_test.go, prepare_test.go and open_test.go: does each launch path (auto start, auto fallback, explicit refusal, full-access skip, OpenLaunch) have coverage, and does any new test duplicate another?"
  - "docs/overview.md and doc.go wording against the code: do they describe auto, the explicit-workspace-write refusal and the machine key as the code implements them?"
  - "The change description's claims, including what it states it could not confirm on OSL-1046 (the AppArmor cause), against the diff."
---

Seed focus for round 1. The decision record holds only the board entry under `## Goal`, so it is the measuring stick.
