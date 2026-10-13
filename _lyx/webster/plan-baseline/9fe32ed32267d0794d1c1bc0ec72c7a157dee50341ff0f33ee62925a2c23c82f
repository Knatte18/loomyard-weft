---
format: 5
approved: true
language: go
---

# Plan: logging-cleanup

Bring lyx's logs to `internal/logger`'s Level policy: an ordinary state logs at Info or Debug once per change, nothing logs per tick or per poll at Info or above without a state change, and no test leaks logger output to stderr.
The plan first gives the logger its test-time discard and a failure-episode helper with the policy text and `PATTERN-log-levels`, then fixes the reed watchdog's resize-failure flood at its cause (a daemon running a replaced binary) and at its log line, then moves batten, orch, shuttle and the loop verbs to the policy, then archives a torn-down pair's logs in the hub, and last sends and handles a notice for every landing.
The Discussion-Review carry-over finding on a dropped landing notice was checked against the code read for card 19: decision D17's next-wake board checks are the recovery the record settled on, and the plan carries them as recorded, adding no scope of its own.

## Card Index

1 — logger-test-discard — under go test the logger's stderr half defaults to io.Discard, and every test restore returns to that default
2 — logger-episode-policy — logger.Episode and EpisodeSet, the three new Level policy rules and PATTERN-log-levels
3 — reed-resize-episode — the resize re-apply failure logs Warn once per episode, Debug while it lasts, Info at recovery
4 — reed-callee-episodes — the reed watch tick's and name-repair loop's callee failures log once per episode
5 — reed-watchdog-retire — the watchdog retires and respawns on a replaced binary, and lock contention exits at Debug
6 — claudeengine-name-read — a missing transcript logs at Debug, any other session-name read failure Warns once per session
7 — battencli-create-log — the create-worktree line drops its duplicate mutations field
8 — battenshed-wait-levels — batten waits log Info once per state with a dedupe key, failed steps Warn, halt Info, notice state Debug
9 — battenshed-read-guards — batten's polled seam failures log once per episode, and repeated attempts log Info once
10 — shuttle-draft-kind — the idle probe and the busy send error carry a draft kind apart from their text
11 — shuttle-read-guards — shuttle's polled failures log once per episode, and a held re-prompt logs Info once
12 — orch-probe-holds — benign probe states agree, disagreements log Info once per episode, draft holds carry no draft text, tick failures log once
13 — shedverbs-read-guards — the loop watchdog's and waiter's read failures log once per episode
14 — loomcli-idle-release-guards — the loop's idle-release wake hook logs a repeating read failure once per episode, and a release at Info
15 — landingshed-pause-read-guard — the parent-lock wait's pause-flag read failure logs once per episode
16 — fabric-log-archive-dir — fabricengine names the hub log archive root
17 — pairteardown-log-archive — pair teardown copies the pair's trace and verify logs into the archive and prunes it to 30 days
18 — land-notice — every landing queues a landed notice, naming the tasks it made ready
19 — orch-stencil-landing — the orch stencil routes every landed notice to After landing and catches a dropped one at its next wake

## Shared Decisions

### The D11 audit: Info-or-above lines in a loop or polled path

The audit swept every loop and polled path in the module, long-lived or bounded, and lists below every Info, Warn or Error line it reaches with its verdict.
"Reaches" follows each loop's per-iteration call graph to its leaves: a line in the loop body, in any callee at any depth the body calls on every iteration, and in a hook, callback or seam the loop's owner hands to a polling loop in another package, such as `shedverbs`' `Spec.Loop.OnWake`, `parentlock`'s `Waiter.Paused` and battenshed's `InnerRunDeps` seams, followed in turn into every package it calls, down to readers that log and skip a record without returning an error.
A site reached only through such a callee or callback names the loop that reaches it.
A line that logs and returns no error is never covered by its caller's guard on the returned error, so it gets its own row.
"Keep" records a line that fires once per state change, once per event, once per bounded attempt or once per run.
"Guard" means `logger.Episode` or `EpisodeSet` (card 2): Warn at the first failure, Debug while it repeats, Info at recovery.

| Site | Line | Verdict | Card |
| --- | --- | --- | --- |
| `reedengine` `Engine.handleWatchOutcome` | `reed: resize re-apply failed` | guard (D3) | 3 |
| `reedengine` `Engine.handleWatchOutcome` | `abandoning this resize event after max attempts`; `told worktree root is gone`/`is back`; `promoting resize watchdog to signal mode` | keep: once per event or state change | — |
| `reedengine` watch loop setup and signal handling | `failed to stat`/`remove resize signal file`, `could not watch the resize signal file`, `invalid watchdog value`, `resize watchdog disabled` | keep: once per watcher start or per signal event | — |
| `reedengine` `Engine.liveBoxLocked` (every watch tick, through `reapplyLayout` and `applyLayoutLockedOpts`, before the box-equality skip) | `failed to query live window size, falling back to configured box`, `malformed live window size answer, falling back to configured box` | guard | 4 |
| `reedengine` `Engine.readBorderTitleRowLocked` (every watch tick that takes no box-equality skip, which a fallback box never takes) | `failed to read back pane-border-status, assuming no title row` | guard | 4 |
| `reedengine` `Engine.keepZoomLocked` (every watch tick on a zoomed window, around `reapplyLayout`) | `failed to unzoom the strands' window before a step`, `failed to list the strands' window after a step`, `failed to re-zoom the strands' window after a step` | guard | 4 |
| `reedengine` `Engine.runResizePinsLocked` (watch tick, through `applyLayoutLockedOpts`; a fallback box takes no box-equality skip, so while the live size query fails the pins run on every tick) | `failed to pin a pane height after the layout apply` | guard, per pane | 4 |
| `reedengine` `Engine.adoptPaneGenerationLocked` (every watch tick through `reapplyLayout` and every name-repair wake through `repairNames`, both via `loadOrInitStateLocked`) | `could not read the live pane generation, leaving persisted pane bindings as they are` | guard | 4 |
| `reedengine` `Engine.adoptPaneGenerationLocked` (same paths; neither saves state, so the in-memory clear is reloaded from disk on every tick until a persisting op runs) | `persisted pane bindings were minted against a different tmux session incarnation, clearing them` | Warn once per change of the recorded and live generation pair, Debug while it repeats | 4 |
| `reedengine` `Engine.loadOrInitStateLocked` (same paths and the same reload) | `cleared strand pane bindings that named a pane another owner already claims` | Warn once per change of the cleared set, Debug while it repeats | 4 |
| `reedengine` `Engine.watchNames` (name-repair loop) | `name repair pass failed` | guard | 4 |
| `reedengine` `Engine.repairNames` (every name-repair wake) | `could not repair pane title` | guard, per strand | 4 |
| `reedengine` `Engine.repairNames` | `repaired pane title` | keep: a repair ends the drift | — |
| `reedengine` `Engine.repairSessionName` (watchdog tick) | `operator activity unreadable, not holding the session rename` | guard | 3 |
| `reedengine` `Engine.repairSessionName` (every name-repair wake while the name drifts: the capture runs before any idleness check, and a failed type leaves the drift) | `could not capture pane for session name repair`, `could not type the session rename` | guard, per strand | 3 |
| `reedengine` `Engine.repairSessionName` | `repaired session name` | keep: a typed rename ends the drift | — |
| `reedcli` `reedCLI.watchdogCmd` | `watchdog lock already held, exiting` | demote to Debug (D4) | 5 |
| `reedcli` `runWatchdogLoop` | `watchdog daemon starting`, `exiting after consecutive idle discovery cycles`, `could not watch the discover signal file`, `could not consume the discover signal`, `reaping orphaned session`, `could not reap orphaned session` | keep: once per run, per signal event or per reaped session | — |
| `shuttleengine/claudeengine` `Claude.SessionNameDrift` (watchdog idle tick) | `could not read the session name` | missing transcript Debug, else Warn once per session (D10) | 6 |
| `battenshed` `reportStuck` and the inner-run wait reports | `producer stuck` | Warn for a failed step, Info once per dedupe key for a wait (D5) | 8 |
| `battenshed` `innerRunProducer.warnHaltEpisode` | `inner shed run halted; waiting for the operator` | Info, once per halt episode (D6) | 8 |
| `battenshed` `innerRunProducer.logSessionStatesAtNotice` | `session state at notice` | Debug (D7) | 8 |
| `battenshed` `lockStep`, `loopLockStep`, `releaseLock`, `parentMovedUnderGate`, `reviewWaitNote`, `reviveDeadDriver`, `driverAlive`, `noticeStep`, `noticeDriverBusy`, `agentActivity`, `arbiterHolds`, `stopReport`, `undecided`, `deliver` | each seam's read or release failure Warn | guard | 9 |
| `battenshed` `innerRunProducer.handedLockStep` | `decision unreadable for the parent lock step; keeping the handed lock` | keep: a failed decision read ends the awaiting arm's Call on the same check through `awaitingStep`'s hard end, and a pause-covered check returns from `lockStep` before the read, so the line never repeats | — |
| `battenshed` `innerRunProducer.releaseLock` | `released the child's parent lock` | demote Warn to Info: a state change, not a failure | 9 |
| `battenshed` `publishRetryGoto`, `publishRetryResume` | `sending the child…`, `resuming the child…`, `inner shed run wait complete` | Info on the first attempt, Debug on retries | 9 |
| `battenshed` `resumeAwaitingChild` | `resuming awaiting inner shed run`, `inner shed run wait complete` | Info on the first attempt, Debug on retries | 9 |
| `battenshed` `resumeAwaitingChild` | `child driver has not parked yet`, `child driver pane is busy` | keep: already once per refusal kind | — |
| `battenshed` `innerRunProducer.coverage`, `pauseRequested` | pause record and pause flag read Warns | keep: already once per distinct error | — |
| `battenshed` `deliver`, `rowNotifier.send` | `notice not delivered`, `notice dropped`, `row notice …` | keep: bounded by `noticeMaxAttempts` | — |
| `battenshed` `primelockwait.go`, `landProducer.acquireLandLock` | `prime lock contended, waiting`, `land lock contended, waiting` | keep: first attempt only | — |
| `battenshed` publish-retry decision, done arm, spawn | publish-retry record Warns, `driver still live after the exit grace`, `unparseable done-seen marker`, `spawning`/`adopting inner shed run` | keep: once per Call or per episode | — |
| `battencli` `childGoto` (battenshed's `Goto` seam, called on every Publish-retry check while the goto is refused) | `sending the loom run to its Publish failure target`, `loom goto wait complete` | demote to Debug: battenshed's own lines around every `Goto` call record the attempt, at Info on the first | 9 |
| `battencli` `battenCLI.wire`'s `Spawn` seam (called on every resume retry through `resumeAwaitingChild` and `publishRetryResume`) | `spawning loom session`, `loom session wait complete` | demote to Debug: battenshed's own lines around every `Spawn` call record the attempt, at Info on the first | 9 |
| `battencli` `readSessionStatesByStrand` (every battenshed activity check, through the `Activity` seam `readAgentActivity`) | `batten: session states unreadable; the notice logs carry none` | keep: `ReadSessionStates` fails only on the run root's `ReadDir`, which `readAgentActivity` has just read without an error through `ReadAgentActivity` on the same root, so a root that stays unreadable fails that earlier read, whose error card 9 guards at `agentActivity`, and this line never runs | — |
| `battencli` `lockIdleBound` | `landing config unreadable; the parent lock's idle bound falls back to its default` | keep: once per `wire`, outside any loop | — |
| `shuttleengine` `operatorActivityHold` (every send poll and orch probe) | `operator activity unreadable, not holding the write` | guard, per runner and strand | 11 |
| `shuttleengine` `Run.probeExpiry` | `gate: expiry probe failed` | guard, per entry | 11 |
| `shuttleengine` `Run.checkLivenessTick` | `capture pane during startup probe` | guard | 11 |
| `shuttleengine` `Run.checkLivenessTick` | `dismiss trust prompt (non-fatal)` | guard | 11 |
| `shuttleengine` `Run.checkLivenessTick` | `persist run started failed` | keep: once, when the run is marked started | — |
| `shuttleengine` `Run.handleGatedBoundary` | `gate: re-prompt not delivered` | demote: Info at the first held attempt, Debug while it stays held | 11 |
| `shuttleengine` `Run.handleGatedBoundary` | `gate: pending send failed, entry stays pending` | guard, per entry | 11 |
| `shuttleengine` `Run.watchDialog` (every wait tick while a dialog latch stands) | `capture pane for the dialog watch failed, retrying on a later tick` | guard | 11 |
| `shuttleengine` `ReadAgentWaits`, `ReadAgentActivity`, `ReadSessionStates` (every wake of `shedverbs`' child watch through loomcli's `readAgentStates` as `Spec.Loop.OnWake`, every idle-watchdog wake through loomcli's `agentActivityFor`, and every battenshed check through battencli's `readAgentActivity` and `readAgentStates` seams) | `skipping an unreadable run record` | guard, per run directory | 11 |
| `shuttleengine` `heldMarkerLive` (through `ReadAgentWaits`) and `ReadWaitMarkers` (through `agentActivityFor` and battencli's `readAgentActivity`) | `skipping an unreadable wait marker` | guard, per marker path | 11 |
| `shuttleengine` `newestTurnEnd` (through `ReadAgentActivity`) | `could not parse a run's events for its activity` | guard, per events path | 11 |
| `shuttleengine` `Run.logSessionState`, `Run.showWait` (every wait tick) | `session state`, `session state disagrees`, the wait mark and wait marker Warns | keep: once per change of state, pair or label | — |
| `shuttleengine` run wait | `turn end held, output files missing`, `turn end waiting on background work`, `background shell past wait min`, `pause record unreadable` | keep: once per turn end, per shell or per distinct error | — |
| `shuttleengine` `Runner` start hold (`run.go`) | `run start held by a pause` | keep: logged by the pause wait's notify, once when the hold begins | — |
| `shuttleengine` API-error hold (`hold.go`) | `automatic continue after an API error not sent`, `continued an API error stall after a machine sleep`, the continue-flag and notified-offset persist Warns, `hold notice failed` | keep: once per held turn end, which the notified offset records | — |
| `orchengine` `Watcher.logIdleStateDisagreement` | `session state disagrees with the idle probe` | Info once per episode, benign pairs agree (D8) | 12 |
| `orchengine` `Watcher.logHeld`, `Watcher.heldTyping` | `injection held`, `injection held by the session` | keep the once-per-change dedupe; draft holds use a constant reason (D9) | 12 |
| `orchengine` `Watcher.Run` | `watcher tick failed` (up to 30 in a row) | guard | 12 |
| `orchengine` `Watcher.orchNetworkOperationInFlight` | `the pause's network operation markers cannot be read` | guard | 12 |
| `orchengine` watcher | `state was bound to another strand mid-tick`, `pointer delivered by the session-start hook`, `resume mark unreadable; the pointer is typed` | keep: once per rebind or per reload pointer step | — |
| `orchengine` `Watcher.typeColor` | `recording the held strand color failed`, `typing the strand color failed` | keep: a failed color is not retried, and a held one retries through `heldTyping`'s once-per-change dedupe | — |
| `orchengine` watcher | `cycle phase stuck`, `session start signal read`, `compaction boundary found`, pause interrupt and pause-record lines, `skill skipped`/`unverified`, `stale cycle request removed` | keep: once per change of reason, per event or per pause | — |
| `shedverbs` `newestActivity` (idle watchdog wake) | `the watchdog could not read the run's agent activity` | guard | 13 |
| `shedverbs` `takeSnapshot` (waiter poll) | `the waiter could not read the loop pid file` | guard | 13 |
| `shedverbs` `watchIdle`, kill path | pause-record read Warn, `asked the idle child to dump its goroutines`, `kills the idle child` | keep: once per run or per kill | — |
| `shedverbs` loop step (`loop.go`) | `loop could not update its pid file`, `loop re-reads an unreadable status file`, `loop re-steps a transient failure` | keep: once per child start, or once until a step succeeds | — |
| `loomcli` `idleReleaseOn` through `ownedLockBranches` (every wake of `shedverbs`' child watch, as `Spec.Loop.OnWake`) | `could not list the parent lock records` | guard | 14 |
| `loomcli` `idleReleaseOn` (every child-watch wake) | `could not read the agents' states for the parent lock's idle rule` | guard | 14 |
| `loomcli` `idleReleaseOn` | `could not release the idle parent lock` | keep: every release attempt restarts the idle watch, so an attempt, failed or not, comes at most once per idle bound | — |
| `loomcli` `idleReleaseOn` | `released the parent lock after every agent waited past the idle bound` | demote Warn to Info: a state change, not a failure | 14 |
| `loomcli` `loopLockExit` through `ownedLockBranches` | `could not list the parent lock records`, `could not settle the parent lock at the loop's exit` | keep: once per loop exit | — |
| `parentreview` gate `closures.holding` | reviewer liveness Warn, `holding the delivery prompt` | keep: once per round and distinct error | — |
| `parentreview` gate `closures.pollAlignment` | `alignment poll failed; delivering to the reviewer` | keep: a failed poll ends the alignment | — |
| `gateslot` pool wait | `gate slot wait start`/`end` | keep: once per wait | — |
| `websterengine` batch polling | — | no Info-or-above line in its poll loops | — |
| `pairteardown` quiet and run-lock waits | `session ended`, board-claim Warns, `ending a driver that is not quiet` | keep: once per teardown | — |
| `gitrepo` push retry | `push refused, retrying` | keep: out of scope, a recoverable retry | — |
| `reedengine` boot loop (`lifecycle.go`) | `boot attempt`, `boot poll has-session check failed`, `tmux server up but session did not materialize`, `zombie boot, reaping socket before retry` | keep: once per bounded boot attempt | — |
| `seatengine` `noticeSender.deliver` | `advisor notice dropped`, `advisor notice never landed before the sender stopped` | keep: once per notice line | — |
| `fabricengine` push loop (`coalesce.go`) | `push rejected, remote diverged; commits left unpushed`, `remote kept refusing the push; commits left unpushed` | keep: once per push | — |
| `landingshed` `PRAlign.wait` | — | no Info-or-above line in its poll loop | — |
| `landingshed` `parentGuard.pauseRequested` (every poll of `parentlock.Store.Acquire`'s unbounded wait, as `Waiter.Paused`) | `could not read the pause flag during the parent lock wait` | guard | 15 |
| `parentlock` `Store.withLock` (every poll of `Store.Acquire` through `TryAcquire`) | `parentlock: release branch lock` | keep: gofrs/flock's `Unlock` returns a failed `LOCK_UN` before closing the descriptor, so the flock stays held by this process and the next poll's blocking `AcquireWriteLock` on a new descriptor waits on it instead of reaching the line again | — |
| `landingshed` `parentGuard.ensure` (`Waiter.OnWait`) | `could not write the parent lock wait record`, the wait mark Warn | keep: once per wait and per change of holder | — |

### Not testable before the deploy

The acceptance criterion that the hub watchdog's trace shows no `resize re-apply failed` line for a session holding a `loom-status` strand holds only after this task's deploy and D18, and is checked by reading that trace then.
The landing report names D18: after the deploy the operator ends each running pre-task `lyx reed watchdog` process once, and the next `reed up`, `attach` or `resume` spawns a daemon that carries card 5's check.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
