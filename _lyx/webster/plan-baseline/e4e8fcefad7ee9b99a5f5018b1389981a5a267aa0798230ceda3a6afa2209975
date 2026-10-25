# Card 14 — innerrun-spawn-hold-yield

**Edit:**
- `internal/battenshed#innerRunProducer.holdSpawnWhileCovered`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#TestInnerRun_OwnerCheck`
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#pollFloor`
- `internal/battenshed#waitOrCancel`
- `internal/battenshed#newInnerRunDeps`
- `internal/battenshed#noticeHarness.producer`
- `internal/battenshed#TestNotice_NilNotifyRunsNoStep`
- `internal/battenshed#TestInnerRun_CancelledDuringASeamErrorReportsTheCancellation`
- `internal/battenshed#TestInnerRun_DoneWaitIsNeitherCountedNorBlockedByTheRunningBudget`
- `internal/battenshed#TestNewInnerRun_FloorsPollInterval`
- `internal/testkit/envkit#nilLegal`
- `internal/shedrecipe#innerRunEntry`
- `internal/battencli#shortPollBattenRecipe`
- `internal/battencli#wireForHub`
- `internal/battencli#stepStepDrivenRunShed_ReturnsOnTheChildsStateChange`
- `internal/battencli#stepRunShedPausedChild_WaitsThenTearsDownOnceDone`

**Delete:**
- `internal/shedrecipe#TestInnerRunEntry_NilSleepIsAccepted`

**Create:**
- `plan:internal/battenshed#TestInnerRun_SpawnHoldYields` -> `func TestInnerRun_SpawnHoldYields(t *testing.T)`

**Uses:**
- `internal/shedengine#Yield`

**Intent:**
Make the spawn hold yield instead of sleeping (D4): a pass at Run-Shed under a covering pause or a standing owner ends at once with the hold reason.
`holdSpawnWhileCovered` reads coverage once, and whenever `ownedBy` stands, covered or not, it first runs the owner check when a notice probe is due by the persisted `lastProbe` (recording the probe), as today's loop runs `ownerStep` on a due probe while `ownedBy` stands, so InnerRun's own owner pause never stops the re-check (D4); an owned-here verdict clears `ownedBy`.
Only then does it end: while a pause covers the slug it writes `held by the pause of <scope> since <paused_at>` to the reason file as today and returns a `Yield` end; while an owner still stands it writes the owner hold reason and yields; with neither it returns nil so `Call` goes on to the spawn in the same pass.
Batten's own pause flag still returns its pause-requested Stuck, and a cancelled context its error.
The hold no longer sleeps, so it has no time bound of its own and the pass stays in flight only through the in-pass waits D4 names.
`Call` returns the hold's end as its result, so the yield, its reason and the memory record's save go out together.
The hold was the last reader of the poll cadence, card 13 having deleted `wait`, so the cadence goes: `InnerRunDeps.Sleep` is deleted with `NewInnerRun`'s `waitOrCancel` default, and the producer's `pollInterval` field with it.
`waitOrCancel` stays for Land's and the row notice's own `Sleep` defaults, its doc naming no InnerRun seam, and `pollFloor`'s doc names the told interval rather than an in-call wait.
`poll_interval_s` stays in the recipe only as a validated key: `innerRunEntry` still reads it and refuses a negative value, and its one effect is `NewInnerRun`'s sub-second floor Warn.
`innerRunEntry`'s doc names no `Env.InnerRun.Sleep`, and `shortPollBattenRecipe`'s doc, which justifies its lowered `poll_interval_s` by the seconds a step spends between checks and by not faking `Env.InnerRun.Sleep`, says instead that it is the recipe copy the tier1 steps build their shed from, its `poll_interval_s` reaching only the floor check.
`envkit`'s `nilLegal` seam inventory loses `InnerRun.Sleep`, so `TestFullEnv_FillsEverySeam` keeps passing.
The tests fill no `Sleep`: `newInnerRunDeps`, `noticeHarness.producer`, `TestNotice_NilNotifyRunsNoStep`, `TestInnerRun_CancelledDuringASeamErrorReportsTheCancellation`, `TestInnerRun_DoneWaitIsNeitherCountedNorBlockedByTheRunningBudget` and `TestNewInnerRun_FloorsPollInterval` drop their fills, a test that moved the child through `fakeClock.Sleep` keeping its own call to it between Calls; battencli's `wireForHub` drops its `InnerRun.Sleep` counter, and `stepStepDrivenRunShed_ReturnsOnTheChildsStateChange` and `stepRunShedPausedChild_WaitsThenTearsDownOnceDone` drop their `InnerRun.Sleep` assignments.
`TestInnerRunEntry_NilSleepIsAccepted` is deleted, since the field it pins is gone and `TestInnerRunEntry_NilNowIsAccepted` already pins that a default env builds.
`TestInnerRun_SpawnHoldYields` (tier0, parallel) rows: a covered absent-status child yields at once with the pause reason in the reason file and no spawn; a covering pause lifted between two Calls: the first Call spawns nothing and yields, and the second, its `Pause` answering uncovered, spawns once.
`TestInnerRun_OwnerCheck` gains rows: with an owner standing and no probe due the pass yields with the owner reason and runs no check; with a probe due and an owned-here verdict the same pass spawns; with an owned-elsewhere verdict it yields and spawns nothing; with an owner pause covering the slug and a probe due, an owned-here verdict clears `ownedBy` while the pass still yields on the pause and spawns nothing.

**ImpactSummary:** One producer helper stops sleeping and its one caller returns its end; with nothing left sleeping, InnerRun loses its `Sleep` seam and poll cadence, and the tests, envkit inventory, recipe entry and tier1 wiring that filled or named the seam follow.
