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
- `internal/dobbyengine#newPassHarness`
- `internal/dobbyengine#runAnAdoptHold`
- `internal/dobbyengine#TestRun_ForeignOwner`
- `internal/dobbyengine#TestOwnedElsewhere`
- `internal/dobbyengine#TestDrivePasses_ExitHandling`
- `internal/dobbyengine/doc.go`
- `internal/beamcli#hubDobbyDeps`
- `internal/loomcli#prePushCheck`
- `internal/loomcli#newPrePushCheck`
- `internal/loomcli#prePushCheck.guard`
- `internal/loomcli#TestPrePushCheckGuard`
- `internal/loomcli#stoppingPrePushCheck`
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#newLockHarness`
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
- `internal/orchcli#startHarness`
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
- `internal/pauserecord/doc.go`
- `internal/machinecli#recoveries`
- `pattern/PATTERN-dobby-runs.md`
- `internal/landingshed#Finalize.adopterFirstRepair`
- `internal/landingshed#parentGuard.release`
- `internal/landingshed#parentGuard.stillHeld`
- `internal/landingshed#TestFinalize_LockRecheckAndLanding`
- `internal/landingshed/doc.go`

**Delete:**
- `internal/dobbyengine#daemon.recheckOwnerPause`
- `internal/beamcli#liftOwnerPause`
- `internal/beamcli#TestLiftOwnerPause_LeavesAnAdoptOrHoldEntryStanding`
- `internal/orchcli#InstallOwnerPause`
- `internal/orchcli#installedOwnerPause`
- `internal/orchcli#ownerPause`
- `internal/orchcli#ownerPauseMu`
- `internal/hibernateengine#fakeWorld.endOwned`

**Uses:**
- `internal/pairteardown#EndSubjectProcesses`
- `internal/boardengine#ReadOwners`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/hibernatecli/doc.go`
- `internal/machinecli/doc.go`

**Intent:** Retire the `owner` pause caller with every writer: a run another machine owns is not paused, the act rule answers `hand_over` and the catch-up hands it over (D5).

dobby: `ownedElsewhere`, run on a pass's owned-elsewhere exit, keeps the owned-elsewhere mark and the owed full pass, releases the parent lock record and posts its one notice, and owes the catch-up (card 21) instead of pausing; the notice reads `<slug>: owned by <owner>; this machine hands it over; way forward: run lym adopt <subject> on this machine to take it over, or on the owner's machine to continue it there`.
`Deps.PauseOwned`, `Deps.LiftOwnerPause` and `Deps.Machine` go, with `recheckOwnerPause`, the probe branch of `drivePasses` that called it, the daemon's `ownerPauses` episodes and `RunState.OwnerPauseFailed`; `passExited` stops clearing that field.
`hubDobbyDeps` stops filling them, and `liftOwnerPause` is deleted with its test.

loom's pre-push check: an entry owned elsewhere stops the push and posts its notice without pausing; `pauseOwned` goes from `prePushCheck` and `newPrePushCheck`.
The stopped push fails the run's step as a stop does today, and the parent dobby, which lists the run's verdict as `hand_over`, owes and runs the catch-up that ends the run's processes and hands it over (card 21).
battenshed's InnerRun: the owner check's owned-elsewhere verdict releases the parent lock, notices and holds the producer as today, but pauses nothing; `InnerRunDeps.PauseOwned`, its default in `NewInnerRun` and battencli's filler in `wire` go, and envkit's `nilLegal` drops `InnerRun.PauseOwned`.
The notice and the hold reason say the run is held, not paused.
The seam tests marking `InnerRunDeps` stay as they are, since they walk the field set they find.

The bring-up (`lym beam start`): `pauseOwnedRuns` ends each owned_elsewhere entry's processes instead of pausing it, so a subject read as owned elsewhere leaves no process of its runs, its orch or its dobby here.
`orchCLI`'s `pauseOwned` field is replaced by `endOwned func(location *lymcwd.Location, slug string, endDobby bool) error`, which `wireCLI` fills with `pairteardown.EndSubjectProcesses(prime, slug, endDobby)`;
a nil seam is skipped and warned as `pauseOwned` is today.
`endDobby` is set when the entry is a beam, read through an `isBeam func(slug string) (bool, error)` seam on `orchCLI` that `wireCLI` fills from `boardengine.ReadOwners(...).Entries[slug].Beam`, since `fabricengine.EntryOutcome` carries no beam flag.
A failed `isBeam` read ends nothing for that entry and warns as a failed end does, the read's error in the "could not be ended" warning, so a beam's dobby is never left running by a guess; a nil `isBeam` reads no entry as a beam.
`InstallOwnerPause`, `installedOwnerPause`, the `ownerPause` and `ownerPauseMu` vars they alone read and write, and `newRoot`'s install go, and `ownerpause.go`'s header comment stops naming the owner pause seam.
The report key `owner_pauses`, `orchcli.BeamUpReport.OwnerPauses` and `beamcli`'s `startEntry.OwnerPauses` keep their names and shape: the slug and owner of each entry whose processes ended.
`pauseOwnedRuns`' two warnings read "<slug> is owned by machine <owner> and was not ended; re-run `lym beam start`, which ends it again" and "<slug> is owned by machine <owner> and could not be ended (<err>); re-run `lym beam start`, which ends it again".
The hibernate wake: `holdRuns` neither pauses nor ends a run owned elsewhere and still names its owner and `lym adopt` in the notice, since the dobby's catch-up after the wake hands it over; `Deps.EndOwned`, its wiring in `hibernatecli`'s `hubDeps` and the fake's `endOwned` go.
After this card no code path writes the `owner` caller; `pauserecord.OwnerCaller` stays declared for the upgrade's first read.
The texts describing the retired pause follow:
`pauserecord/doc.go` drops the owner pause's lifter and the sentence saying the dobby's owner-pause lift and the hibernate wake judge `OwnerCaller`, keeping the parent-lock exemption's;
`machinecli`'s `recoveries`, which `lym machine name` prints, drops the line saying the dobby's probe lifts an owner pause, and the next line names a pause left standing as one `lym tender resume <slug>` lifts, without the probe;
`PATTERN-dobby-runs.md`'s Adopt pause rule drops the dobby's owner lift;
and landingshed's texts (`adopterFirstRepair`'s repair note, `parentGuard`'s release causes and halt, and `landingshed/doc.go`) stop naming an owner pause: a lock released for another machine's ownership is that ownership's hand-over.

Tests, tier0 unless their file names another:
`TestOwnedElsewhere` asserts no pause, the lock release, one notice and the owed catch-up, and drops the owner-pause lift subtests.
`TestDrivePasses_ExitHandling`, `newLoopHarness`, `newPassHarness`, `runAnAdoptHold` and `TestRun_ForeignOwner` drop the `PauseOwned`, `LiftOwnerPause` and `Machine` fields they set.
`TestFinalize_LockRecheckAndLanding` asserts the halt's reworded causes and way forward.
`TestPrePushCheckGuard` asserts an owned-elsewhere stop pauses nothing, and `stoppingPrePushCheck` drops `pauseOwned`.
`TestInnerRun_OwnerCheck` asserts no pause call, and `newLockHarness` drops `PauseOwned`.
`TestStart_ReconcilesTheHubFirst` and `startHarness.runWith` assert the processes end through `endOwned`, with the dobby end set for a beam entry and unset for a task, the two reworded warnings, and a failed `isBeam` read ending nothing and warning; `startHarness`'s test-local fields keep their names, and their comment describes the end in place of the owner pause.
`TestSequence` asserts a run owned elsewhere gets no entry and no end, and is named in the notice.
Each package doc drops the owner pause, `beamcli/doc.go` and `loomcli/doc.go` included.

**ImpactSummary:** Removes the owner pause from five callers across seven packages and the root install; one seam field leaves InnerRunDeps, dobbyengine.Deps and hibernateengine.Deps each, and orchCLI swaps pauseOwned for endOwned and gains isBeam.
