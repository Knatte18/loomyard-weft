# Card 3 — watchdog-verb-keeps-stderr

**Edit:**
- `internal/reedcli#reedCLI.watchdogCmd`
- `internal/reedcli#validateWatchdogFlags`
- `internal/reedcli/doc.go`
- `internal/reedcli/watchdog_integration_test.go`

**Intent:**
Decision D2, the verb half: `lyx reed watchdog` no longer calls `logger.SetOutput(io.Discard)`, so a foreground run honours `-v`, `-vv`, `LYX_LOG_LEVEL` and `LYX_LOG_FILE` like every other verb.
It still points the durable sink at the hub logs directory first, through `logger.SetDurableSinkDir(fabricengine.HubLogsDir(hubPath))`, before taking the lock.
Drop the `io` import with the call.
A detached daemon's stderr half reaches nothing, since its nil stdio is the null device and card 2 keeps `LYX_LOG_FILE` out of its environment; its Debug trace reaches only the durable sink, under that sink's cap.

The verb's `Long` help replaces the sentence claiming its stdio is discarded with the operator's two routes to a Debug trace of the daemon: run it in the foreground with `-vv`, whose Debug lines then go to stderr or the `LYX_LOG_FILE` the operator set for that run; or, while no daemon runs for the hub, run `lyx reed up`, `attach` or `resume` with `LYX_LOG_LEVEL=debug` set, so the detached daemon that verb spawns starts at Debug and writes its trace to the hub logs directory.
It also states that a daemon already running keeps the level it started with, since a new spawn finds the hub lock held and exits.
The comment in `validateWatchdogFlags` that reasons from "its child's stderr is discarded" is reworded to the spawn being best-effort and detached with no stderr anyone watches.

`internal/reedcli/doc.go` gains a paragraph on the daemon's logging: the durable sink in the hub logs directory, Debug reaching it at Debug verbosity, and the two routes above in one sentence.

Test: the `DaemonLockAndLogs` step of `TestWatchdogDaemon` captures the logger's stderr half with `logcapture.CaptureVerbose` before starting the first daemon command and asserts the captured buffer carries the daemon's "watchdog daemon starting" line, which the discarded stderr half could never show; its step comment, which documents the sink-then-discard-then-lock ordering, is rewritten to sink-then-lock with stderr kept.
The step is already `tmux`-tier and does not call `t.Parallel`, so the process-global capture is safe there.
Grep for the old help text ("its own stdio is discarded") shows no test asserting it.

**ImpactSummary:** One verb stops discarding its stderr half and rewords its help; the discovery loop, the lock and the durable sink wiring are unchanged, and only a foreground run shows more output.
