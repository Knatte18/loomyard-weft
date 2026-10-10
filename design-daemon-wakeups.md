# Daemons and wake-ups: event-driven waits, a timely orch reload, recorded revival order

`daemon-wakeups` · Task · bug, enhancement, reed, orch, shuttle · running · Describe

## Goal

lyx's long-lived processes wait cheaply and wake on time:

- Every lyx daemon waits on events or second-scale backoff polls, pinned by a guard, with `lyx reed watchdog` brought under the rule first.
- The orch watcher sends its post-compact resume steps right after the compaction, not minutes later behind a disagreeing pane idle probe.
- reed revives sessions in their recorded start order, not by a file mtime that can move.

The three parts below are the merged notes, each kept as written.

## Part 1: daemon idle cost

### What happened

On 2026-10-08 the machine ran slow with five runs in flight, and `lyx reed watchdog` showed up at 14% CPU in a snapshot.
Over 43 hours it used 36 minutes of CPU, about 1.4% of one core on average, for a daemon that mostly waits.
Its loop polls for the resize signal file every 100 ms (`watchdogSignalTick` in `internal/reedengine/watchdog.go`) around the clock, and runs a tmux `list-sessions` round trip every discovery cycle.

### Proposal

A lyx daemon costs close to nothing while nothing happens.
More daemons are coming (`creel` and others, operator 2026-10-08), so this is a rule for all of them, not a fix for one:

- A daemon waits on events, not on a timer: file-system notification (inotify on Linux, its equivalents on Windows and macOS through one shared package) for signal files, and tmux hooks or a control-mode client for tmux state, where those exist.
- Where it has to poll, its interval is in seconds, not tenths of a second, and backs off while idle.
- A guard test or a PATTERN invariant pins the rule, so a new daemon cannot ship with a sub-second poll unannounced.

`lyx reed watchdog` is the first daemon brought under the rule; measure its idle CPU before and after.


## Part 2: the orch reload lands late (#481)

Imported from [issue #481](https://github.com/Knatte18/loomyard/issues/481).

After a distill on 2026-10-10, the watcher's resume steps (`/color`, `/reload-plugins`, the "read role.md and the orch note" prompt) reached the orch session almost three minutes after the compaction, after the operator had already held four exchanges with the un-reloaded session.
The operator reports the same lateness after earlier compactions as well; `lifecycle-waits` (done) and #356, #390 fixed neighbouring cases, not this one.

### Timeline (UTC, from the orch transcript and `.lyx/orch/watch.log`)

- 04:07:10 handoff note requested (`handoff-20261010T040710Z.md`), written, turn ended.
- 04:08:25 compaction boundary written to the transcript.
- 04:08:45 the operator's next message arrives; the session works on it with no role, skills or color reloaded.
- 04:08:26, 04:09:53, 04:10:24, 04:10:57: at each of the orch's turn ends the watcher logs `orch: session state disagrees with the idle probe` with `idle=false state=asking cause=awaiting-input`, and injects nothing.
- 04:11:09 the phase enters `resuming` (`state.json`), and 04:11:12 `/reload-plugins` and the resume prompt land.

So the session's own state said it was waiting for input at every turn end, but the pane idle probe said not idle, and the watcher waited for the probe.

### Expected

- The resume steps follow the compaction boundary at once, before the operator's next message is taken, or are queued so they run ahead of it.
- A turn end whose session state is `asking`/awaiting input is enough to inject; a disagreeing pane probe does not hold the reload back for minutes.
- The watcher logs why each resume step waited, so the delay can be read from `watch.log` without the transcript.


## Part 3: reed revival order

Revival orders sessions by the mtime of `seed.json`, which re-materialization can move.
Record an explicit start time in the run record and order revival by it.
