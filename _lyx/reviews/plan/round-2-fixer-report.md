# Plan round 2 — fixer report

## Changes

- **F1** (fixed as proposed): card 5's **Intent:** now says `watchdogDownWait` and `watchdogDownPoll` are declared in `internal/reedcli/watchdog.go` beside `watchdogLockFileName`, with `watchdogDownPoll`'s line carrying `//lyx:one-shot bounded lock poll of at most watchdogDownWait`, the `watchLockRetryInterval` form, because `cmd/lyx/daemonwakeups_test.go`'s `unboundedWaitFiles` lists `watchdog.go`.
  Premise checked: `internal/reedcli/watchdog.go` is in `unboundedWaitFiles` and `watchdogLockFileName` is declared there.
- **F2** (fixed as proposed): card 3's **Edit:** list gains `internal/reedengine#DiscoverSignalFileName` and `internal/reedengine#Geometry`, and its **Intent:** says both comments are extended to name `Engine.Down` writing `DiscoverStopMarker` when it ends the server.
  Premise checked: `internal/reedengine/watchdog.go` and `internal/reedengine/geometry.go` name only a cold session boot as the writer.
- **F3** (fixed as proposed): card 4's prescribed `watchdogHubIdleCycles` comment now names a daemon whose watch failed to open and a transient list-sessions error, which `sessionsAreIdle` counts as idle, and drops the non-last `down`.
  Premise checked: `sessionsAreIdle` returns `!(err == nil && len(names) > 0)`.

`lyx loom validate-plan` passes after the edits.

## Disputed

None.

## Deferred

None.
