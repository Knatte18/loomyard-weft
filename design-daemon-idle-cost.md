# Daemons wait on events, not tight polls: near-zero CPU while idle

`daemon-idle-cost` · Note · enhancement, reed

## What happened

On 2026-10-08 the machine ran slow with five runs in flight, and `lyx reed watchdog` showed up at 14% CPU in a snapshot.
Over 43 hours it used 36 minutes of CPU, about 1.4% of one core on average, for a daemon that mostly waits.
Its loop polls for the resize signal file every 100 ms (`watchdogSignalTick` in `internal/reedengine/watchdog.go`) around the clock, and runs a tmux `list-sessions` round trip every discovery cycle.

## Proposal

A lyx daemon costs close to nothing while nothing happens.
More daemons are coming (`creel` and others, operator 2026-10-08), so this is a rule for all of them, not a fix for one:

- A daemon waits on events, not on a timer: file-system notification (inotify on Linux, its equivalents on Windows and macOS through one shared package) for signal files, and tmux hooks or a control-mode client for tmux state, where those exist.
- Where it has to poll, its interval is in seconds, not tenths of a second, and backs off while idle.
- A guard test or a PATTERN invariant pins the rule, so a new daemon cannot ship with a sub-second poll unannounced.

`lyx reed watchdog` is the first daemon brought under the rule; measure its idle CPU before and after.
