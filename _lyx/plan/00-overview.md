---
format: 5
approved: false
language: go
---

# Plan: logging-cleanup

Bring lyx's logs to `internal/logger`'s Level policy: an ordinary state logs at Info or Debug once per change, nothing logs per tick or per poll at Info or above without a state change, and no test leaks logger output to stderr.
The plan first gives the logger its test-time discard and a failure-episode helper with the policy text and `PATTERN-log-levels`, then fixes the reed watchdog's resize-failure flood at its cause (a daemon running a replaced binary) and at its log line, then moves batten, orch, shuttle and the loop verbs to the policy, then archives a torn-down pair's logs in the hub, and last sends and handles a notice for every landing.
The Discussion-Review carry-over finding on a dropped landing notice was checked against the code read for card 16: decision D17's next-wake board checks are the recovery the record settled on, and the plan carries them as recorded, adding no scope of its own.

## Card Index

1 — logger-test-discard — under go test the logger's stderr half defaults to io.Discard, and every test restore returns to that default
2 — logger-episode-policy — logger.Episode and EpisodeSet, the three new Level policy rules and PATTERN-log-levels
3 — reed-resize-episode — the resize re-apply failure logs Warn once per episode, Debug while it lasts, Info at recovery
4 — reed-watchdog-retire — the watchdog retires and respawns on a replaced binary, and lock contention exits at Debug
5 — claudeengine-name-read — a missing transcript logs at Debug, any other session-name read failure Warns once per session
6 — battencli-create-log — the create-worktree line drops its duplicate mutations field
7 — battenshed-wait-levels — batten waits log Info once per state with a dedupe key, failed steps Warn, halt Info, notice state Debug
8 — battenshed-read-guards — batten's polled seam failures log once per episode, and repeated attempts log Info once
9 — shuttle-draft-kind — the idle probe and the busy send error carry a draft kind apart from their text
10 — shuttle-read-guards — shuttle's polled failures log once per episode, and a held re-prompt logs Info once
11 — orch-probe-holds — benign probe states agree, disagreements log Info once per episode, draft holds carry no draft text, tick failures log once
12 — shedverbs-read-guards — the loop watchdog's and waiter's read failures log once per episode
13 — fabric-log-archive-dir — fabricengine names the hub log archive root
14 — pairteardown-log-archive — pair teardown copies the pair's trace and verify logs into the archive and prunes it to 30 days
15 — land-notice — every landing queues a landed notice, naming the tasks it made ready
16 — orch-stencil-landing — the orch stencil routes every landed notice to After landing and catches a dropped one at its next wake

## Shared Decisions

### The D11 audit: Info-or-above lines in a loop or polled path

The audit covered the long-lived loops and polled paths: reed's watch loop and the hub watchdog (`internal/reedengine/watchloop.go`, `internal/reedcli/watchdog.go`), batten's Run-Shed wait and the rows after it (`internal/battenshed`), the orch watcher (`internal/orchengine`), shuttle's run wait, verified send and idle probe (`internal/shuttleengine`), the loop watchdog and waiter (`internal/shedverbs/loopwatchdog.go`, `loopwait.go`), the parent-review gate (`internal/parentreview/gate.go`), the gate slot pool (`internal/gateslot/pool.go`), webster's batch polling (`internal/websterengine`), and pair teardown's quiet and run-lock waits (`internal/pairteardown`).
Every Info, Warn or Error line in those loops is listed below with its verdict; a line at Info or above in none of them is outside the audit, and so is a line that fires once per state change, once per event or once per run, which "keep" records.
"Guard" means `logger.Episode` or `EpisodeSet` (card 2): Warn at the first failure, Debug while it repeats, Info at recovery.

| Site | Line | Verdict | Card |
| --- | --- | --- | --- |
| `reedengine` `Engine.handleWatchOutcome` | `reed: resize re-apply failed` | guard (D3) | 3 |
| `reedengine` `Engine.handleWatchOutcome` | `abandoning this resize event after max attempts`; `told worktree root is gone`/`is back`; `promoting resize watchdog to signal mode` | keep: once per event or state change | — |
| `reedengine` watch loop setup and signal handling | `failed to stat`/`remove resize signal file`, `could not watch the resize signal file`, `invalid watchdog value`, `resize watchdog disabled` | keep: once per watcher start or per signal event | — |
| `reedengine` `Engine.repairSessionName` (watchdog tick) | `operator activity unreadable, not holding the session rename` | guard | 3 |
| `reedengine` `Engine.repairSessionName` | `could not capture pane for session name repair`, `could not type the session rename`, `repaired session name` | keep: a capture or type failure is retried only while the name still drifts and the pane is idle, and a repair ends the drift | — |
| `reedcli` `reedCLI.watchdogCmd` | `watchdog lock already held, exiting` | demote to Debug (D4) | 4 |
| `reedcli` `runWatchdogLoop` | `watchdog daemon starting`, `exiting after consecutive idle discovery cycles`, `could not watch the discover signal file`, `could not consume the discover signal`, `reaping orphaned session`, `could not reap orphaned session` | keep: once per run, per signal event or per reaped session | — |
| `shuttleengine/claudeengine` `Claude.SessionNameDrift` (watchdog idle tick) | `could not read the session name` | missing transcript Debug, else Warn once per session (D10) | 5 |
| `battenshed` `reportStuck` and the inner-run wait reports | `producer stuck` | Warn for a failed step, Info once per dedupe key for a wait (D5) | 7 |
| `battenshed` `innerRunProducer.warnHaltEpisode` | `inner shed run halted; waiting for the operator` | Info, once per halt episode (D6) | 7 |
| `battenshed` `innerRunProducer.logSessionStatesAtNotice` | `session state at notice` | Debug (D7) | 7 |
| `battenshed` `lockStep`, `handedLockStep`, `loopLockStep`, `releaseLock`, `parentMovedUnderGate`, `reviewWaitNote`, `reviveDeadDriver`, `driverAlive`, `noticeStep`, `noticeDriverBusy`, `agentActivity`, `arbiterHolds`, `stopReport`, `undecided`, `deliver` | each seam's read or release failure Warn | guard | 8 |
| `battenshed` `innerRunProducer.releaseLock` | `released the child's parent lock` | demote Warn to Info: a state change, not a failure | 8 |
| `battenshed` `publishRetryGoto`, `publishRetryResume` | `sending the child…`, `resuming the child…`, `inner shed run wait complete` | Info on the first attempt, Debug on retries | 8 |
| `battenshed` `resumeAwaitingChild` | `resuming awaiting inner shed run`, `inner shed run wait complete` | Info on the first attempt, Debug on retries | 8 |
| `battenshed` `resumeAwaitingChild` | `child driver has not parked yet`, `child driver pane is busy` | keep: already once per refusal kind | — |
| `battenshed` `innerRunProducer.coverage`, `pauseRequested` | pause record and pause flag read Warns | keep: already once per distinct error | — |
| `battenshed` `deliver`, `rowNotifier.send` | `notice not delivered`, `notice dropped`, `row notice …` | keep: bounded by `noticeMaxAttempts` | — |
| `battenshed` `primelockwait.go`, `landProducer.acquireLandLock` | `prime lock contended, waiting`, `land lock contended, waiting` | keep: first attempt only | — |
| `battenshed` publish-retry decision, done arm, spawn | publish-retry record Warns, `driver still live after the exit grace`, `unparseable done-seen marker`, `spawning`/`adopting inner shed run` | keep: once per Call or per episode | — |
| `shuttleengine` `operatorActivityHold` (every send poll and orch probe) | `operator activity unreadable, not holding the write` | guard, per runner and strand | 10 |
| `shuttleengine` `Run.probeExpiry` | `gate: expiry probe failed` | guard, per entry | 10 |
| `shuttleengine` `Run.checkLivenessTick` | `capture pane during startup probe` | guard | 10 |
| `shuttleengine` `Run.handleGatedBoundary` | `gate: re-prompt not delivered` | demote: Info at the first held attempt, Debug while it stays held | 10 |
| `shuttleengine` run wait | `turn end held, output files missing`, `turn end waiting on background work`, `background shell past wait min`, `pause record unreadable` | keep: once per turn end, per shell or per distinct error | — |
| `shuttleengine` `Runner` start hold (`run.go`) | `run start held by a pause` | keep: logged by the pause wait's notify, once when the hold begins | — |
| `shuttleengine` API-error hold (`hold.go`) | `automatic continue after an API error not sent`, `continued an API error stall after a machine sleep`, the continue-flag and notified-offset persist Warns, `hold notice failed` | keep: once per held turn end, which the notified offset records | — |
| `orchengine` `Watcher.logIdleStateDisagreement` | `session state disagrees with the idle probe` | Info once per episode, benign pairs agree (D8) | 11 |
| `orchengine` `Watcher.logHeld`, `Watcher.heldTyping` | `injection held`, `injection held by the session` | keep the once-per-change dedupe; draft holds use a constant reason (D9) | 11 |
| `orchengine` `Watcher.Run` | `watcher tick failed` (up to 30 in a row) | guard | 11 |
| `orchengine` `Watcher.orchNetworkOperationInFlight` | `the pause's network operation markers cannot be read` | guard | 11 |
| `orchengine` watcher | `state was bound to another strand mid-tick`, `pointer delivered by the session-start hook`, `resume mark unreadable; the pointer is typed` | keep: once per rebind or per reload pointer step | — |
| `orchengine` `Watcher.typeColor` | `recording the held strand color failed`, `typing the strand color failed` | keep: a failed color is not retried, and a held one retries through `heldTyping`'s once-per-change dedupe | — |
| `orchengine` watcher | `cycle phase stuck`, `session start signal read`, `compaction boundary found`, pause interrupt and pause-record lines, `skill skipped`/`unverified`, `stale cycle request removed` | keep: once per change of reason, per event or per pause | — |
| `shedverbs` `newestActivity` (idle watchdog wake) | `the watchdog could not read the run's agent activity` | guard | 12 |
| `shedverbs` `takeSnapshot` (waiter poll) | `the waiter could not read the loop pid file` | guard | 12 |
| `shedverbs` `watchIdle`, kill path | pause-record read Warn, `asked the idle child to dump its goroutines`, `kills the idle child` | keep: once per run or per kill | — |
| `parentreview` gate `closures.holding` | reviewer liveness Warn, `holding the delivery prompt` | keep: once per round and distinct error | — |
| `gateslot` pool wait | `gate slot wait start`/`end` | keep: once per wait | — |
| `websterengine` batch polling | — | no Info-or-above line in its poll loops | — |
| `pairteardown` quiet and run-lock waits | `session ended`, board-claim Warns, `ending a driver that is not quiet` | keep: once per teardown | — |
| `gitrepo` push retry | `push refused, retrying` | keep: out of scope, a recoverable retry | — |

### Not testable before the deploy

The acceptance criterion that the hub watchdog's trace shows no `resize re-apply failed` line for a session holding a `loom-status` strand holds only after this task's deploy and D18, and is checked by reading that trace then.
The landing report names D18: after the deploy the operator ends each running pre-task `lyx reed watchdog` process once, and the next `reed up`, `attach` or `resume` spawns a daemon that carries card 4's check.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
