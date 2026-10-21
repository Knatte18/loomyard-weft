---
round: 2
focus:
  - "internal/reedcli/watchdog.go, runWatchdogLoop timer arm while `unwatched` is set: does a consume failure that persists across wakes log at the level `internal/logger`'s Level policy sets for a repeating failure, and does a later successful consume end the episode?"
  - "internal/reedcli/watchdog_test.go, TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping: does a test drive the closed-event-channel path and check that a stop is still read on the timer?"
---

The focus targets the places the round-1 fixes will touch.
