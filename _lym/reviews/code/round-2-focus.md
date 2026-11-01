---
round: 2
focus:
  - "In internal/ideengine/spawn.go, check what directory Spawn passes to placeWindow for a prime spawn, whose launch path is the hub .code-workspace file, and whether desktophook.AfterWindow (internal/desktophook/window.go) ever receives a file path as cmd.Dir. Settle whether the window command runs for the prime's window."
  - "In internal/desktophook/window.go, check whether the window_wait_ms sleep and the command timeout and wait delay can be driven by a test without real sleeps, and whether a test pins that the wait precedes the command and that a hanging command returns promptly with an error."
  - "In internal/ideengine/spawn.go, check that OpenTask and a failing CodeLauncher (in both Spawn and OpenTask) are covered by a test that asserts the window command runs once or not at all respectively."
---

Round 2 re-checks the fixes for the gating findings and the coverage gaps.
