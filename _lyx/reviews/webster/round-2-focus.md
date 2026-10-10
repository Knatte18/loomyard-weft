---
round: 2
focus:
  - "internal/reedcli/watchdog_integration_test.go, the step that boots a pair through reed and expects the watchdog to discover it through the discover signal: does the step fail if the signal wake in runWatchdogLoop (internal/reedcli/watchdog.go, the event branch that calls onSignal and resets the timer) is removed, or can the backed-off cadence alone satisfy its time window?"
  - "internal/orchengine/watcher.go, probeIdle, logHeld and logTyped: after a probe passes and a non-reload injection is typed, is a later hold with the same reason logged again, and is the hold memory (lastHeld, heldSince) reset when a probe passes?"
  - "internal/orchengine/watcher.go, State.Stuck written by a hold: after a watcher restart that clears lastHeld in initCursor, can a passing probe clear a Stuck text a hold wrote, or does it stay on lyx orch status until the next enter?"
---

Focus restates the sites each question depends on; it caps no severity and pre-states no verdict.
