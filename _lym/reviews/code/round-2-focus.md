---
round: 2
focus:
  - "internal/reedcli/watchdog.go runWatchdogLoop and internal/reedcli/oomnotice.go reportPaneOOMKills: settle whether the pane OOM check can block the watchdog's discovery, reaping or binary-check steps, through Engine.Status's blocking op lock, the journalctl read or the creel post."
  - "internal/reedcli/oomnotice.go planPaneOOMNotices and paneOOMNoticeLine: settle whether a kill is attributed only to a pane capped into its own lym-pane scope, and whether a notice names only journal entries no earlier notice named."
  - "internal/reedcli/oomnotice.go planPaneOOMNotices and reportPaneOOMKills: settle whether a failed creel post leaves the kill untold so a later pass retells it."
  - "internal/reedcli and internal/reedengine tests: settle whether a test drives a new pane OOM kill through to a creel post from REED, and whether a pane that cannot be capped is shown to launch with a Warn naming the cause."
---

The fixer's changes to the OOM notice path are new text; judge them as the diff leaves them.
