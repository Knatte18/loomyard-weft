---
round: 2
focus:
  - "internal/shuttleengine/codexengine/sandbox.go, runSandboxProbe and launchSandboxMode: settle which probe failures reach the degrade-to-danger-full-access branch under auto, and whether a timeout, a missing binary or a signalled probe is refused instead of degraded, under auto and workspace-write alike."
  - "internal/shuttleengine/codexengine/open_test.go and prepare_test.go: settle whether OpenLaunch's workspace-write refusal and the probe's bin, home and dir arguments are each pinned by a test row, and whether each new row for a non-refusal probe failure is a coverage gap or duplicates an existing row."
---

The focus names where the round-1 fixes land and the question each must settle.
