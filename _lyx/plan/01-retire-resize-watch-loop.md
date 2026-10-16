# Card 1 — Retire the per-worktree resize watch loop

**Delete:**
- `internal/reedengine/reapply.go`
- `internal/reedengine/reapply_test.go`
- `internal/reedengine#TestWatchState`
- `internal/reedengine#watchDefaultTimingMatchesTheSixConstants`
- `internal/reedengine#watchPollWakeChangedPerOutcome`
- `internal/reedengine#watchStateSingleSignalWaitsThenApplies`
- `internal/reedengine#watchStateCoalescesABurstIntoOneApply`
- `internal/reedengine#watchStateSignalInsideQuietRestartsIt`
- `internal/reedengine#watchStateSignalDuringInFlightApplySchedulesOneFollowUp`
- `internal/reedengine#watchStateSucceededClearsTheOwedApply`
- `internal/reedengine#watchStateFailedEscalatesAndCaps`
- `internal/reedengine#watchStateStreakResetsOnSuccess`
- `internal/reedengine#watchStateStreakResetsOnFreshSignal`
- `internal/reedengine#watchStateDeferredChangesNothing`
- `internal/reedengine#watchStateFreshSignalAfterExhaustedStreakReArms`
- `internal/reedengine#watchdogTestTiming`
- `internal/reedengine#newWatchLoopTestEngine`
- `internal/reedengine#feedWatch`
- `internal/reedengine#newFeedWatch`
- `internal/reedengine#feedWatch.open`
- `internal/reedengine#feedWatch.opened`
- `internal/reedengine#feedWatch.send`
- `internal/reedengine#startWatchLoop`
- `internal/reedengine#eventually`
- `internal/reedengine#TestWatchLoop_ParkedWhenNotEnabled`
- `internal/reedengine#TestWatchLoop_StaleSignalFileRemovedAtStart`
- `internal/reedengine#TestWatchLoop_PollModeByDefault`
- `internal/reedengine#waitForPromotion`
- `internal/reedengine#TestWatchLoop_SignalMode`
- `internal/reedengine#TestWatchLoop_UndecidedProbeDoesNotGuess`
- `internal/reedengine#TestWatchLoop_TakeEffectBoundary`
- `internal/reedengine#TestWatchLoop_FailuresNeverKillTheLoop`
- `internal/reedengine#TestWatchLoop_DeferralCostsNoBudget`
- `internal/reedengine#TestWatchLoop_PollModeGoesDormantOnVanishedWorktreeRoot`
- `internal/reedengine#TestWatchLoop_RecoversFromDormancyToItsPriorMode`
- `internal/reedengine#TestWatchLoop_NonSentinelFailureDoesNotGoDormant`
- `internal/reedengine#TestWatchLoop_OpenerFailureStaysInPollMode`
- `internal/reedengine/watchdog_integration_test.go`
- `internal/reedengine#watchPlan`
- `internal/reedengine#watchPlanWait`
- `internal/reedengine#watchPlanApply`
- `internal/reedengine#watchTiming`
- `internal/reedengine#watchDefaultTiming`
- `internal/reedengine#watchState`
- `internal/reedengine#newWatchState`
- `internal/reedengine#watchState.Signal`
- `internal/reedengine#watchState.Plan`
- `internal/reedengine#watchState.Succeeded`
- `internal/reedengine#watchState.Failed`
- `internal/reedengine#watchState.Deferred`
- `internal/reedengine#watchMode`
- `internal/reedengine#watchModePoll`
- `internal/reedengine#watchModeSignal`
- `internal/reedengine#watchModeDormant`
- `internal/reedengine#oneShot`
- `internal/reedengine#oneShot.arm`
- `internal/reedengine#oneShot.stop`
- `internal/reedengine#oneShot.fired`
- `internal/reedengine#Engine.Watch`
- `internal/reedengine#pollWakeChanged`
- `internal/reedengine#Engine.consumeResizeSignal`
- `internal/reedengine#Engine.watchLoop`
- `internal/reedengine#Engine.handleWatchOutcome`
- `internal/reedengine#watchdogDebounceQuiet`
- `internal/reedengine#watchdogPollCycle`
- `internal/reedengine#watchdogPollCeiling`
- `internal/reedengine#watchdogRetryBaseDelay`
- `internal/reedengine#watchdogMaxAttempts`
- `internal/reedengine#watchdogDormantCycle`
- `internal/reedcli#watchdogWindowSize`
- `internal/reedcli#watchdogResizeWindow`
- `internal/burlercli#TestReedUpSeam_WatcherLifecycle`
- `internal/webstercli#TestReedUpSeam_WatcherLifecycle`

**Edit:**
- `internal/burlercli#burlerCLI`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/burlercli#burlerCLI.runCmd`
- `internal/burlercli#TestRunCmd`
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#websterCLI.runCmd`
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/webstercli#TestRecoverBatchCmd_FabricSyncAndReedBootWayForward`
- `internal/webstercli#TestBringUpDisposition_RecoverAndRun`
- `internal/reedengine#countLogLines`
- `internal/reedcli#enterSession`
- `internal/reedcli#watchedSession`
- `internal/reedcli#runWatchdogLoop`
- `internal/reedcli#watchdogHubDiscoveryCycle`
- `internal/reedcli#watchdogTiming`
- `internal/reedcli#TestWatchdogDaemon`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`

**Intent:** D6's watch-loop half: the per-worktree layout re-apply and the loop that drives it go, and the per-hub daemon keeps its per-session watch as the name repair loop alone.

Delete `reapply.go` whole (`ReapplyResult`, `hookArrayEntries`, `hookInstalledLocked`, `reapplyLayout`) and, from `watchloop.go`, every member of the resize loop: the signal-mode decision state, the modes, `oneShot`, `Engine.Watch`, `pollWakeChanged`, `consumeResizeSignal`, `watchLoop` and `handleWatchOutcome`.
`FileWatchOpener` and `OpenFileWatch` stay in `watchloop.go`, since `runWatchdogLoop` opens the discover signal file through them; the file header is rewritten to name only that file-event seam.
The loop's fixed timings in `watchdog.go` go with it; `watchdogOption`, `NextWakeCadence`, `DiscoverSignalFileName` and the resize-signal and hook-command members stay until card 2, which removes the hook array that still writes them.
The `watchdog.go` file header drops the loop's timings from its inventory.

In `enterSession` the goroutine running `eng.Watch` goes; the `WatchNames` goroutine and the `watchdog: off` early return stay, so `watchdog: off` still keeps that worktree's name repair off.
The doc comments of `watchedSession`, `runWatchdogLoop`, `watchdogHubDiscoveryCycle` and `watchdogTiming` stop naming `Engine.Watch` and its loop; `runWatchdogLoop`'s note that a flipped `watchdog:` value takes effect when the session re-enters stays, now about name repair.

The two standalone producers drop their in-process resize watcher too: the `reedUp` field of `burlerCLI` and of `websterCLI` becomes `func() error` (a struct field, so no arrow), its doc dropping the watcher and the `watch` parameter's recover-batch asymmetry.
Each `wireStandalone` closure only runs `reedEngine.Up()`; `burlerCLI.runCmd`, `websterCLI.runCmd` and `websterCLI.recoverBatchCmd` call `c.reedUp()`.

In both reed templates the `watchdog` key stays, and its comment is rewritten to name the name repair it gates in the per-hub daemon, with the resize self-heal, watch loop and `window-resized` wording dropped.

Tests: `reapply_test.go`, `watchloop_test.go` and `internal/reedengine/watchdog_integration_test.go` pin only the retired loop and go whole.
`countLogLines`, the one helper in `watchloop_test.go` that tests outside it call (`TestRepairNames_SessionName`, `TestCalleeFailuresLogOncePerEpisode`, `TestEnsureShellWindow`, `TestLoadOrInitStateLocked`), moves unchanged into `calleefailures_test.go` before `watchloop_test.go`'s other members are deleted and the emptied file is removed, so the package's tests keep compiling.
`TestWatchdogDaemon` loses its `ResizeAppliesOnlyToThatWorktree` subtest, and `watchdogWindowSize` and `watchdogResizeWindow`, which only that subtest calls, go; its discovery, reap and name-repair subtests stay as they are, so the daemon's name repair stays covered.
Both `TestReedUpSeam_WatcherLifecycle` tests pin only the retired watcher and go; `TestRunCmd`'s bring-up row, `TestRecoverBatchCmd_FabricSyncAndReedBootWayForward` and `TestBringUpDisposition_RecoverAndRun` assert that run and recover-batch each call the seam once, with no `watch` argument to check.
`TestWatchdogTemplateDefault_BothGOOS` keeps pinning the key's default and is unchanged.

**ImpactSummary:** removes one exported engine method (`Engine.Watch`) whose three callers are `enterSession` in `internal/reedcli` and the `wireStandalone` closures of `internal/burlercli` and `internal/webstercli`; the daemon's discovery, reap and name repair are untouched.

**Commit:** 1: reed: retire the per-worktree resize watch loop
