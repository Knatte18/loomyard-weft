---
format: 5
approved: false
language: go
---

# Plan: reed down ends its watchdog; strand panes resolve lyx to the spawning binary

Two reed defects from the creel stage 1 re-test.
#679: `lyx reed down` of a hub's last session now asks the hub watchdog to stop through the discover signal file, the daemon exits at once on a stop-woken idle cycle, re-checks for sessions after releasing its lock, retires on a replaced binary with a successor only for a live hub, and `down` waits, bounded, for the lock to come free.
#680: the `LYX_BIN` key moves to `internal/agentname`, and the Claude engine's `CLAUDE_ENV_FILE` re-prepends `LYX_BIN`'s directory so an agent's Bash tool resolves bare `lyx` to the spawning binary after the operator's rc.
The package docs and the three sandbox suites' pre-condition bullets follow.

## Card Index

1 — lyx-bin-env-key — Declare the LYX_BIN env key in agentname and read it in reed's prelude
2 — env-file-prepends-lyx-bin — The Claude engine's env file re-prepends LYX_BIN's directory, per OS
3 — down-writes-discover-stop — Engine.Down writes a stop marker and reports ending the server
4 — watchdog-stop-and-recheck — The daemon exits on a stop, re-checks on exit and respawns only for a live hub
5 — down-waits-for-watchdog — The down verb polls the watchdog lock, bounded, after ending the server
6 — sandbox-suite-preconditions — The sandbox suites name the shell their lyx check runs in

## Shared Decisions

- The stop request is the content `reedengine.DiscoverStopMarker` in the existing discover signal file; no new file or watch is added, and a cold boot's truncating touch turns a pending stop back into a discover.
  The marker is written atomically, a temp file renamed onto the signal path, so the daemon's watch never sees the file empty before the marker lands.
- The daemon's session lister is a seam (`runWatchdogLoop`'s `list` parameter, `reedCLI.watchdogListSessions`), so the stop, re-check and replaced-ordering rows run hermetically.

## verify:

```bash
go build ./cmd/lyx
lyx gate test ./internal/agentname ./internal/reedengine ./internal/reedcli ./internal/shuttleengine/claudeengine ./cmd/lyx
lyx gate test --tags integration ./internal/agentname ./internal/reedengine ./internal/reedcli ./internal/shuttleengine/claudeengine ./cmd/lyx
lyx gate test --tags tmux ./internal/reedengine ./internal/reedcli
```
