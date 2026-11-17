# Card 23 — retire-owner-pause-callers

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#daemon`
- `internal/dobbyengine#RunState`
- `internal/dobbyengine#daemon.ownedElsewhere`
- `internal/dobbyengine#daemon.noticeOwnedElsewhere`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#daemon.passExited`
- `internal/dobbyengine#newLoopHarness`
- `internal/dobbyengine#TestOwnedElsewhere`
- `internal/dobbyengine#TestDrivePasses_ExitHandling`
- `internal/dobbyengine/doc.go`
- `internal/beamcli#hubDobbyDeps`
- `internal/loomcli#prePushCheck`
- `internal/loomcli#newPrePushCheck`
- `internal/loomcli#prePushCheck.guard`
- `internal/loomcli#TestPrePushCheckGuard`
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#innerRunProducer.ownedElsewhere`
- `internal/battenshed#innerRunProducer.noticeOwnedElsewhere`
- `internal/beamcli/doc.go`
- `internal/loomcli/doc.go`
- `internal/battenshed#TestInnerRun_OwnerCheck`
- `internal/battenshed/doc.go`
- `internal/testkit/envkit#nilLegal`
- `internal/battencli#battenCLI.wire`
- `internal/orchcli#orchCLI`
- `internal/orchcli#wireCLI`
- `internal/orchcli#orchCLI.pauseOwnedRuns`
- `internal/orchcli#startHarness.runWith`
- `internal/orchcli#TestStart_ReconcilesTheHubFirst`
- `internal/orchcli/doc.go`
- `cmd/lym#newRoot`
- `internal/hibernateengine#Deps`
- `internal/hibernateengine#holdRuns`
- `internal/hibernateengine#fakeWorld`
- `internal/hibernateengine#fakeWorld.deps`
- `internal/hibernateengine#TestSequence`
- `internal/hibernateengine/doc.go`
- `internal/hibernatecli#hubDeps`

**Delete:**
- `internal/dobbyengine#daemon.recheckOwnerPause`
- `internal/beamcli#liftOwnerPause`
- `internal/beamcli#TestLiftOwnerPause_LeavesAnAdoptOrHoldEntryStanding`
- `internal/orchcli#InstallOwnerPause`
- `internal/orchcli#installedOwnerPause`
- `internal/hibernateengine#fakeWorld.endOwned`

**Uses:**
- `internal/pairteardown#EndSubjectProcesses`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/hibernatecli/doc.go`

**Intent:** Retire the `owner` pause caller with every writer: a run another machine owns is not paused, the act rule answers `hand_over` and the catch-up hands it over (D5).

dobby: `ownedElsewhere`, run on a pass's owned-elsewhere exit, keeps the owned-elsewhere mark and the owed full pass, releases the parent lock record and posts its one notice, and owes the catch-up (card 21) instead of pausing; the notice reads `<slug>: owned by <owner>; this machine hands it over; way forward: run lym adopt <subject> on this machine to take it over, or on the owner's machine to continue it there`.
`Deps.PauseOwned`, `Deps.LiftOwnerPause` and `Deps.Machine` go, with `recheckOwnerPause`, the probe branch of `drivePasses` that called it, the daemon's `ownerPauses` episodes and `RunState.OwnerPauseFailed`; `passExited` stops clearing that field.
`hubDobbyDeps` stops filling them, and `liftOwnerPause` is deleted with its test.

loom's pre-push check: an entry owned elsewhere stops the push and posts its notice without pausing; `pauseOwned` goes from `prePushCheck` and `newPrePushCheck`.
The stopped push fails the run's step as a stop does today, and the parent dobby, which lists the run's verdict as `hand_over`, owes and runs the catch-up that ends the run's processes and hands it over (card 21).
battenshed's InnerRun: the owner check's owned-elsewhere verdict releases the parent lock, notices and holds the producer as today, but pauses nothing; `InnerRunDeps.PauseOwned`, its default in `NewInnerRun` and battencli's filler in `wire` go, and envkit's `nilLegal` drops `InnerRun.PauseOwned`.
The notice and the hold reason say the run is held, not paused.
The seam tests marking `InnerRunDeps` stay as they are, since they walk the field set they find.

The bring-up (`lym beam start`): `pauseOwnedRuns` ends each owned_elsewhere entry's processes through `pairteardown.EndSubjectProcesses(prime, slug, endDobby)` instead of pausing it, with `endDobby` set when the entry is a beam, so a subject read as owned elsewhere leaves no process of its runs, its orch or its dobby here; `InstallOwnerPause`, `installedOwnerPause`, the `pauseOwned` field and `newRoot`'s install go.
The hibernate wake: `holdRuns` neither pauses nor ends a run owned elsewhere and still names its owner and `lym adopt` in the notice, since the dobby's catch-up after the wake hands it over; `Deps.EndOwned`, its wiring in `hibernatecli`'s `hubDeps` and the fake's `endOwned` go.
After this card no code path writes the `owner` caller; `pauserecord.OwnerCaller` stays declared for the upgrade's first read.

Tests, tier0 unless their file names another:
`TestOwnedElsewhere` asserts no pause, the lock release, one notice and the owed catch-up, and drops the owner-pause lift subtests.
`TestDrivePasses_ExitHandling` and `newLoopHarness` drop the owner-pause seams.
`TestPrePushCheckGuard` asserts an owned-elsewhere stop pauses nothing.
`TestInnerRun_OwnerCheck` asserts no pause call.
`TestStart_ReconcilesTheHubFirst` and `startHarness.runWith` assert the processes end through the seam that replaces the pause, with the dobby end set for a beam entry and unset for a task.
`TestSequence` asserts a run owned elsewhere gets no entry and no end, and is named in the notice.
Each package doc drops the owner pause, `beamcli/doc.go` and `loomcli/doc.go` included.

**ImpactSummary:** Removes the owner pause from five callers across seven packages and the root install; one seam field leaves InnerRunDeps, dobbyengine.Deps and hibernateengine.Deps each.
