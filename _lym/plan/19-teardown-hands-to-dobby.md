# Card 19 — teardown-hands-to-dobby

**Edit:**
- `internal/battencli#teardownPushSeams` -> `func teardownPushSeams(location *lymcwd.Location, slug string, locateTask func() (*lymcwd.Location, error), syncOptions func() fabricengine.SyncOptions, start func(runpusher.Target) (bool, error)) (push func(context.Context) error, level func() (bool, error))`
- `internal/battencli#battenCLI.wire`
- `internal/battenshed#worktreeTeardownProducer.Call`
- `internal/battenshed#NewWorktreeTeardown`
- `internal/battenshed/doc.go`
- `internal/beamcli#tearDownLeftover`
- `internal/beamcli#TestTearDownLeftovers_RemovesALandedAndACancelledPairAndTheirWorkspaceFolders`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.Run`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown#Teardown.EndForCancel`
- `internal/pairteardown#HandOver`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/pairteardown#TestEndForCancel`
- `internal/pairteardown_test#TestPushRun_HandsOverOnATakeover`
- `internal/battencli#TestTeardownPushSeams_PushesOffLetTheRowFinishWithoutTouchingThePair`
- `internal/runpusher/doc.go`
- `cmd/lym#unboundedWaitFiles`

**Delete:**
- `internal/runpusher/pushuntillanded.go`
- `internal/battenshed#maxTeardownPushRounds`
- `internal/battenshed#TestWorktreeTeardown_LevelThatNeverReadsTrueIsStuckAfterTheBound`
- `internal/battencli#TestTeardownPushSeams_PushBringsAPresentPairsTipsToOrigin`

**Create:**
- `plan:internal/pairteardown#ErrLandingStands` -> `var ErrLandingStands = errors.New("pairteardown: the run's landing entry stands")`
- `plan:internal/battenshed#TestWorktreeTeardown_AFalseLevelEndsDoneWithThePairStanding` -> `func TestWorktreeTeardown_AFalseLevelEndsDoneWithThePairStanding(t *testing.T)`
- `plan:internal/battencli#TestTeardownPushSeams_PushWakesThePairsPusher` -> `func TestTeardownPushSeams_PushWakesThePairsPusher(t *testing.T)`
- `plan:internal/fabriccli_test#TestCommand_RemoveRefusesWhileALandingStands` -> `func TestCommand_RemoveRefusesWhileALandingStands(t *testing.T)`

**Uses:**
- `internal/beamcli/doc.go`
- `internal/pairteardown/doc.go`
- `internal/fabriccli/doc.go`
- `internal/battenshed#TeardownDeps`
- `internal/battenshed#teardownCallRecorder`
- `internal/fabricengine#TipsOnOrigin`
- `plan:internal/pushledger#HeldRecordsCommit`
- `plan:internal/pushledger#OwedTips`
- `plan:internal/pushledger#Update`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** D13 and D11: Worktree-Teardown pushes nothing itself and hands an unpushed pair to the dobby, and no teardown removes a pair while its run's landing entry stands.

`teardownPushSeams` takes the pusher start as a seam, which `battenCLI.wire` fills with `runpusher.Start`.
Its `Push` sets the pair target's push-now mark through `pushledger.Update` and starts or wakes its pusher through `start`, and returns.
Its `Level` reads true only when `fabricengine.TipsOnOrigin` holds both tips and the run has no standing landing entry.
With git or pushes off it sets no mark, reads no tracking ref, reads level and the pair is removed as today.
`worktreeTeardownProducer.Call` makes one push and one level read: level removes the pair as today, and a false level ends the row Done with the pair standing, so the row waits on no network and loops nowhere;
`maxTeardownPushRounds` and its Stuck go, and `NewWorktreeTeardown`'s doc and `internal/battenshed/doc.go`'s paragraph on the bounded push rounds say so. The `TeardownDeps` fields stay.
`PushUntilLanded` loses its last caller and goes with `internal/runpusher/pushuntillanded.go`; `cmd/lym#unboundedWaitFiles` and `internal/runpusher/doc.go` drop it.

The dobby's `tearDownLeftover` removes a landed run's pair only once `TipsOnOrigin` holds both tips and no landing entry of the run stands; until then it leaves the pair for a later tick and reports nothing failed.

`Teardown` gains a `landingStands func(slug string) (bool, error)` read, which `New` fills from the ledger's landings of the run; a nil read, as a hand-built `Teardown` leaves it, reads as none standing.
`Teardown.Run`, before `EndSession`, `RemovePair`, before its snapshot, and `EndForCancel`, before it ends any process, refuse while the run's landing entry stands, with an error wrapping `ErrLandingStands` and touching nothing:
no process is ended, no board entry is marked done, no records commit is pushed and no origin branch is deleted.
Every removal goes through them, so `lym fabric remove` with or without `--force` or `--remote`, `lym batten cancel`, Worktree-Teardown and the dobby's teardown alike refuse.
Its way forward names the owed parent push: wait for the pusher, or run the step the parent's refused-push record names.
`HandOver`, reached from a rejected push that meets another machine's takeover, archives the tips as today and, while the run's landing entry stands, skips the pair removal and returns `ErrHandedOver` naming that the pair stays until its landing is dropped;
the parent pusher's drop (card 11) then removes the entry, and the dobby's handed-over leftover teardown removes the pair.
A dropped landing (card 11) removes the entry before any removal and leaves its records commit as the pair entry's `HeldRecords`, so the pusher's hand-over and a later `lym fabric remove` pass the check, while their pushes still stop short of the held commit (card 9).
After a successful removal `RemovePair` removes the pair target's ledger entry, its `HeldRecords` with it, so a removed, handed-over or cancelled-away pair's entry goes (D3) and the hold goes with the pair.
Bound: the refusal blocks only the pair of a run whose landing push is owed; it lifts when the parent push lands or the landing is dropped.

Tests.
`TestTeardown_CallSequence`, tier0, adding under a second to one tier0 run, gains rows over a fake `landingStands`: a standing landing refuses `Run` with `ErrLandingStands` and records no call, with and without `Force` and `Remote`; a successful removal leaves no ledger entry of the pair under `t.TempDir()`.
`TestEndForCancel`, tier0, gains a row: a standing landing refuses before any process is ended.
`TestPushRun_HandsOverOnATakeover`, tier1, gains a row: with the run's landing entry standing, the hand-over returns `ErrHandedOver` and leaves the pair standing.
`TestCommand_RemoveRefusesWhileALandingStands`, tier1, over a hub fixture with a pair, its pushed tips and a landing entry in the ledger, runs `lym fabric remove` plain, with `--force` and with `--remote`:
each refuses naming the owed parent push, the pair stands, origin's board entry is not `done`, and origin's records branch lacks the held commit.
`TestWorktreeTeardown_AFalseLevelEndsDoneWithThePairStanding`, tier0, pins one push, one level read, Done and no removal, which also covers an unreachable origin; a level read true removes once.
`TestTeardownPushSeams_PushWakesThePairsPusher`, tier1, over a fake `start`, pins that `Push` writes the push-now mark and hands `start` the pair's target, and that `Level` reads false while both tips are on origin but the run's landing entry stands.
`TestTeardownPushSeams_PushesOffLetTheRowFinishWithoutTouchingThePair` also asserts no mark is written and `start` is never called (D11).
`TestTearDownLeftovers_RemovesALandedAndACancelledPairAndTheirWorkspaceFolders`, tier1, gains rows: a landed pair whose tips origin lacks, and one whose landing entry stands, are left standing; with both on origin and no entry it is removed.

**ImpactSummary:** Changes Worktree-Teardown's push and level steps, the dobby's landed-pair removal, the hand-over's removal and the teardown composite's entries; every removal route now meets one more refusal.
