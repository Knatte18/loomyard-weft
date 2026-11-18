# Card 17 — The orch watcher switches the orch's account and restarts it

**Create:**
- `plan:internal/orchengine#OrchRestart` -> `type OrchRestart struct`
- `plan:internal/orchengine#Watcher.WatchRestart` -> `func (w *Watcher) WatchRestart(restart OrchRestart)`
- `plan:internal/orchengine#TestWatcher_UsageLimitSwitchRestartsTheOrch` -> `func TestWatcher_UsageLimitSwitchRestartsTheOrch(t *testing.T)`
- `plan:internal/orchengine#TestWatcher_RefusedResumeStartsTheOrchFresh` -> `func TestWatcher_RefusedResumeStartsTheOrchFresh(t *testing.T)`
- `plan:internal/orchcli#runnerSession.SwitchAccountOnLimit` -> `func (s runnerSession) SwitchAccountOnLimit(guid string, turnEnd shuttleengine.Event) (bool, error)`
- `plan:internal/orchcli#runnerSession.RefusedResume` -> `func (s runnerSession) RefusedResume(guid string, turnEnd shuttleengine.Event) (bool, error)`
- `plan:internal/orchcli#orchCLI.relaunchOrch` -> `func (c *orchCLI) relaunchOrch(st orchengine.State, fresh bool) (string, error)`
- `plan:internal/orchcli#TestRelaunchOrch_RestartsOverTheLiveLimitedStrand` -> `func TestRelaunchOrch_RestartsOverTheLiveLimitedStrand(t *testing.T)`

**Extends:**
- `internal/orchengine#Session`
- `internal/orchengine#Watcher`
- `internal/orchengine#fakeSession`
- `internal/orchengine/doc.go`
- `internal/orchcli/doc.go`

**Edit:**
- `internal/orchengine#Watcher.tick`
- `internal/orchcli#orchCLI.startOrch`
- `internal/orchcli#orchCLI.watchCmd`

**Uses:**
- `internal/shuttleengine#Runner.SwitchAccountOnLimit`
- `internal/shuttleengine#Runner.RefusedResume`
- `internal/orchengine#recordUsageLimit`
- `internal/orchengine#Watcher.tickUsageLimit`
- `internal/orchengine#limitedWatchEnv`
- `internal/orchcli#orchCLI.launchOrch`
- `internal/orchcli#orchCLI.withdrawFirst`
- `internal/orchcli#orchCLI.registerOrchProcess`
- `internal/orchcli#orchCLI.saveFreshLaunch`
- `internal/orchcli#TestStart_DeadStrandRemovedBeforeStart`

**Intent:**
D7's orch half and D6's orch refused resume.
`Session` gains `SwitchAccountOnLimit(guid, turnEnd)` and `RefusedResume(guid, turnEnd)`, which orchcli's `runnerSession` answers through the runner's methods (cards 13 and 16).
`OrchRestart` holds two told capabilities, `Resume func(State) (State, error)` and `Fresh func(State) (State, error)`, each given the watcher's current state and returning the state saved on the new strand, set through `WatchRestart`; a watcher told none keeps today's behaviour.

In `Watcher.tick`, where a newest turn end read idle reads as a usage limit, the watcher first asks `SwitchAccountOnLimit`; on true it calls `Resume` instead of recording the limit and waiting, and carries on over the new strand from the state the restart saved.
On false, or an error (logged at Warn), it waits the limit out as today through `recordUsageLimit` and `tickUsageLimit`.
Where a newest turn end is an API error that `RefusedResume` reads as a refused resume, it calls `Fresh`.
A failed restart has already removed the old strand, so it is logged at Warn naming the way forward `lym orch start`, and the watcher ends as it does over a dead orch strand.

`orchCLI.relaunchOrch(st, fresh)` is the body of `startOrch`'s `StartRelaunch` case, moved out unchanged, so `startOrch` calls it and then spawns the watcher; it runs directly, skipping `DecideStart` and the wait for the watcher to go, since the live limited strand and the calling watcher would decide that the orch is already running.
It withdraws the orch's pending mail, removes the old orch strand (ending the still-live limited process), launches through `launchOrch` with the session resumed, or with the recorded session id cleared when `fresh` is set so `launchOrch` launches fresh, registers the new process as the worktree's orch, and saves the state reset onto the new strand GUID; it spawns no second watcher and returns the new GUID.
`watchCmd` tells the watcher closures that call `relaunchOrch(st, false)` and `relaunchOrch(st, true)` over the state the tick passes them.

Tests are tier0:
`TestWatcher_UsageLimitSwitchRestartsTheOrch`, on `limitedWatchEnv` with `fakeSession` scripted to switch: the watcher calls `Resume` once and records no usage-limit wait; scripted not to switch, it holds the orch and wakes it after the reset as `TestWatcher_UsageLimitHoldsTheOrchAndWakesItAfterTheReset` pins; a `Resume` that fails ends the watcher with a Warn naming `lym orch start`; after a switch, the next tick reads events from the new strand GUID at the saved state's cursor.
`TestWatcher_RefusedResumeStartsTheOrchFresh`: an API-error turn end `RefusedResume` answers true for calls `Fresh` once; one it answers false for calls neither.
`TestRelaunchOrch_RestartsOverTheLiveLimitedStrand`, on the fakes of `TestStart_DeadStrandRemovedBeforeStart` with the old strand live: afterwards reed holds no old orch strand, the saved state names the new strand GUID, the new process is registered as the worktree's orch, the starter was asked to resume the recorded session, no watcher was spawned; with `fresh` the starter launched with no session id.

**ImpactSummary:** The watcher's limit branch gains a switch ahead of its wait; `startOrch`'s relaunch branch moves into a method it still calls; the watch command wires two closures.

**Commit:** 17: orch: switch the orch's account on a limit and restart it, fresh on a refused resume
