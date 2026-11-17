# Card 9 — `lym adopt` runs in the foreground

**Create:**
- `plan:internal/adoptcli#runForeground` -> `func runForeground(deps adoptDeps, req request, steps io.Writer) (outcome, error)`
- `plan:internal/adoptcli#holdStands` -> `func holdStands(deps adoptDeps, identity machinename.Identity, subjects []string) error`
- `plan:internal/adoptcli#movedRun` -> `func movedRun(hold boardengine.Hold, registry boardengine.Registry, identity machinename.Identity, subject, slug string, takeover *shedengine.Takeover) bool`
- `plan:internal/adoptcli#takeOverRuns` -> `func takeOverRuns(deps adoptDeps, hold boardengine.Hold, owners boardengine.Owners, identity machinename.Identity, adopted []string, rootAdopted bool, outcomes map[string]*slugOutcome) error`

**Edit:**
- `internal/adoptcli#request`
- `internal/adoptcli#slugOutcome`
- `internal/adoptcli#outcome`
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#movedRuns`
- `internal/adoptcli#addError` -> `func addError(slug, rerun string, err error) string`
- `internal/adoptcli#ownerWriteError`
- `internal/adoptcli#runBeamSteps`
- `internal/adoptcli#heldBeamWayForward` -> `func heldBeamWayForward(deps adoptDeps, here string, owners boardengine.Owners, slug string) string`
- `internal/adoptcli#deployStale`
- `internal/adoptcli#validateRequest`
- `internal/adoptcli#callerDeps`
- `internal/adoptcli#hubCallerDeps`
- `internal/adoptcli#callFlags`
- `internal/adoptcli#runAdopt`
- `internal/adoptcli#hereRequest`
- `internal/adoptcli#runArchives`
- `internal/adoptcli#Command`
- `internal/adoptcli#adoptFields`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli/doc.go`
- `internal/adoptcli#world`
- `internal/adoptcli#newWorld`
- `internal/adoptcli#world.deps`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli#callerFakes`
- `internal/adoptcli#TestRunAdopt`
- `internal/adoptcli#TestAdoptFields`
- `internal/adoptcli#TestRunArchives`
- `internal/adoptcli#AdoptAs` -> `func AdoptAs(prime *lymcwd.Location, machine func() (string, error), subjects []string, takeHold bool) (Outcome, error)`
- `internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver`
- `internal/fixtries/doc.go`
- `internal/fixtries_test#TestReadWrite`
- `internal/dobbyengine#upgradeHub.deps`
- `cmd/lym#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`
- `pattern/PATTERN-dobby-runs.md`
- `PATTERN.md`
- `docs/overview.md`

**Delete:**
- `internal/adoptcli/steps.go`
- `internal/adoptcli/follow.go`
- `internal/adoptcli/pause.go`
- `internal/adoptcli/runreport.go`
- `internal/adoptcli/worker_test.go`
- `internal/adoptcli/follow_test.go`
- `internal/adoptcli/runreport_test.go`
- `internal/adoptcli#branchStop`
- `internal/adoptcli#sideBolt`
- `internal/adoptcli#sideRecords`
- `internal/adoptcli#pausedRun`
- `internal/adoptcli#suiteStart`
- `internal/adoptcli#suiteMessage`
- `internal/adoptcli#reconcileWarnings`
- `internal/adoptcli#sharedStopText`
- `internal/adoptcli#stoppedBranches`
- `internal/adoptcli#ownedElsewhere`
- `internal/adoptcli#takeOverSlug`
- `internal/adoptcli#resumeSlug`
- `internal/adoptcli#noteStanding`
- `internal/adoptcli#wouldStop`
- `internal/adoptcli#stepRun`
- `internal/adoptcli#outcomePart`
- `internal/adoptcli#runSteps`
- `internal/adoptcli#stepFunc`
- `internal/adoptcli#seedOutcomes`
- `internal/adoptcli#stepRun.seedOutcomes`
- `internal/adoptcli#stepRun.refuseBeforeSteps`
- `internal/adoptcli#stepRun.beginPart`
- `internal/adoptcli#stepRun.slot`
- `internal/adoptcli#stepRun.endPart`
- `internal/adoptcli#stepRun.recordStep`
- `internal/adoptcli#stepRun.stop`
- `internal/adoptcli#stepRun.save`
- `internal/adoptcli#mergePart`
- `internal/adoptcli#mergeSlugs`
- `internal/adoptcli#workerDeps`
- `internal/adoptcli#hubWorkerDeps`
- `internal/adoptcli#tryAdoptLock`
- `internal/adoptcli#request.recorded`
- `internal/adoptcli#requestOf`
- `internal/adoptcli#standingVerdict`
- `internal/adoptcli#verdictFresh`
- `internal/adoptcli#verdictFollow`
- `internal/adoptcli#verdictResume`
- `internal/adoptcli#judgeStanding`
- `internal/adoptcli#mergeRequest`
- `internal/adoptcli#rejudges`
- `internal/adoptcli#runWorker`
- `internal/adoptcli#splitSubjects`
- `internal/adoptcli#reconcileWire`
- `internal/adoptcli#partWire`
- `internal/adoptcli#decodePart`
- `internal/adoptcli#envelopeOf`
- `internal/adoptcli#workerOutName`
- `internal/adoptcli#workerArgs`
- `internal/adoptcli#workerCommand`
- `internal/adoptcli#spawnWorker`
- `internal/adoptcli#discard`
- `internal/adoptcli#workerCmd`
- `internal/adoptcli#driveWorker`
- `internal/adoptcli#reconcilePushLockWait`
- `internal/adoptcli#reconcileFunc`
- `internal/adoptcli#reconcileSeam`
- `internal/adoptcli#hubOnlyPaused`
- `internal/adoptcli#suiteArgs`
- `internal/adoptcli#startSuite`
- `internal/adoptcli#startCheckWithTiers`
- `internal/adoptcli#adoptGateTiers`
- `internal/adoptcli#ensureCheck`
- `internal/adoptcli#TestDiscard`
- `internal/adoptcli#TestWorkerArgs`
- `internal/adoptcli#stepsThrough`
- `internal/adoptcli#writeAdoptRecord`
- `internal/adoptcli#requireNoOrchArg`
- `internal/adoptcli#TestHubOnlyPaused`
- `internal/adoptcli#TestReconcileSeamWaitsOnAPushLock`
- `internal/adoptcli#TestSpawnersStartNothingUnderTest`
- `internal/adoptcli#TestEnsureCheck`
- `internal/adoptcli#TestAdoptGateTiers`
- `internal/adoptcli#TestStartCheckWithTiers`
- `internal/adoptcli#world.workerDeps`
- `internal/adoptcli#world.pausedScopes`
- `internal/adoptcli#world.stopBranch`
- `internal/adoptcli#recordOf`
- `internal/adoptcli#workerPID`
- `internal/adoptcli#workerTicks`
- `internal/adoptcli#workerTime`
- `internal/adoptcli#pauseTime`
- `internal/adoptcli#testSuite`
- `internal/adoptcli_test#holdOwnerPause`
- `internal/fixtries#Write`

**Uses:**
- `internal/adoptcli#namedEntries`
- `internal/adoptcli#normalizedRequest`
- `internal/adoptcli#projectOf`
- `internal/adoptcli#checkSubjects`
- `internal/adoptcli#refuseStandingMove`
- `internal/adoptcli#stopWith`
- `plan:internal/adoptcli#runHoldCheck`
- `plan:internal/adoptcli#hubHoldCheckDeps`
- `internal/adoptcli#hereSubject`
- `internal/adoptcli#collect`
- `internal/adoptcli#checkAlive`
- `plan:internal/boardengine#WriteAdopt`
- `plan:internal/boardengine#AdoptRequest`
- `plan:internal/boardengine#HoldRefusal`
- `internal/boardengine#PullBolt`
- `plan:internal/fabricengine#TakeOverRun`
- `plan:internal/fabricengine#OriginParentRunStatus`
- `internal/fabricengine#RegenerateHubWorkspace`
- `internal/fabricengine#Topology.Add`
- `internal/battencli#HubCatchUp`
- `plan:internal/battencli#HubRunSlugs`
- `internal/dobbyengine#CatchUp`
- `internal/orchcli#BeamUp`
- `internal/fixtries#Read`

**Intent:**
`lym adopt <subject>...` runs every step in the caller's process (D6) and prints `adopt: <step> done` on stderr as each ends; nothing is detached and no record is written, so a re-run goes on from the tracked state.
Every way forward the steps word names the re-run over the subjects of the hold, `lym adopt <hold's subjects>`, since a re-run of fewer subjects is refused as another set.

`request` drops `KeepLocal` and `DropLocal` and gains `TakeHold`; `validateRequest` drops their clash and refuses the take-hold flag with `--all`: a hold names its subjects, so the take-hold re-run names them; way forward: read them in `lym beam status` (card 6) and name them. The refusal text names the flag only as "the take-hold flag" so no agent reads its spelling, though only the operator can pass it.
`validateRequest` and `normalizedRequest` move to adopt.go, and worker.go, steprun.go and caller.go's worker half go; `stopError` and `stopWith` stay for the deploy and the owner-write errors.
adopt.go's and deps.go's file comments describe the foreground steps, the hold and the seams, with no worker, adopt pause, reconcile, tries file or check start.

`adoptDeps` seams become: `Machine`, `Identity`, `ReadOwners`, `PullBolt`, `ParentOf`, `RunRecord` (local, for the anchor-change refusal), `OriginRun(slug, parent)` over `OriginParentRunStatus`, `LocalPair`, `AddPair`, `WriteAdopt`, `TakeOver(slug, subject)` over `TakeOverRun`, `CatchUp` over `dobbyengine.CatchUp(battencli.HubCatchUp(prime, HubRunSlugs))`, `MoveInProgress`, `DeployState`, `RunDeploy`, `Workspace` over `RegenerateHubWorkspace`, `HoldCheck(first string, noOrch bool)` over `runHoldCheck(hubHoldCheckDeps(prime, os.Stdout, noOrch))` with `BeamUp` nil, `BeamUp`, `LandingHeld`, `PrimeName`, `HubPath`, `Now`.
`hubDepsAs` keeps its signature: the request's `--no-orch` reaches `HoldCheck` at call time.
The reconcile, live-pair, drop-local, owner-write, pause, tries-reset and suite-start seams go.

`runForeground`:
0. Validates the request, reads this machine's identity, and refuses a standing anchor change under the subjects (`refuseStandingMove`) before any write.
1. `WriteAdopt` with the subjects, `TakeHold` and `Now`. A `*HoldRefusal` or registration refusal ends the adopt with its own way forward; `ownerWriteError` words an unreachable origin. Under this machine's same-set hold the hold stays as it stands (`Kept`).
2. `holdStands` pulls Bolt and re-reads the hold before each of steps 2 to 7; once the hold is gone or names another machine it stops, naming the subjects' owner, says the adopt moved to that machine or its hold cleared so nothing more runs here, and starts nothing.
   Its last read is before the check: steps 7 and 8 clear the hold themselves on green, so no re-read follows them.
   An adopted beam with no local pair gets it from origin through `AddPair` (`addError` words a refused add, naming the hold's re-run).
   `movedRuns` now judges each open task or beam entry under the subjects by its record at origin (`OriginRun` from its parent's records branch), not done.
   `movedRun` reads a run as moved when its origin record holds a takeover entry naming another machine, or when it holds no takeover entry and the subject's `PriorOwner` in the hold does not read as this machine.
   A run whose origin takeover entry names this machine (an earlier pass of this adopt took it), a run whose subject's `PriorOwner` reads as this machine with no foreign takeover entry, and the hold's `FixTask` are not moved:
   steps 2 and 3 skip them and step 4 pushes their owed commits, which leaves a fix task's branches and processes untouched on a re-run under this machine's own `adopt-fix` hold.
   The hold keeps `PriorOwner` across a re-run, so an adopt killed after step 1 still moves the runs it took and leaves alone the ones already taken.
   `takeOverRuns` adds each moved run's missing pair from origin through `AddPair`, then calls `TakeOver` (fetch, archive, reset, takeover entry pushed with its retry, fast-forward, hand-over mark dropped); a `TakeOver` error ends the adopt naming the run, the error and the hold's re-run.
   A beam under an adopted root that keeps its own owner on another machine is reported as owned elsewhere and not taken over.
3. Step 3 is inside `TakeOver`; it prints after the last moved run.
4. `CatchUp` over the whole hub. A failing catch-up ends the adopt with its error and the hold's re-run, once origin answers when origin was unreachable; nothing past it runs, since the deploy and the check must read origin's state.
5. The deploy (`deployStale`) when root is a subject, as today; its stop text says the hold stands, so no run goes on under the old lym, in place of the retired adopt pause.
6. `Workspace`; a failure is a warning naming `lym fabric reconcile` in the prime, and the adopt goes on.
7. and 8. `HoldCheck` over the first subject: the gate runs in each subject's worktree with its output on the terminal; green clears the hold and drops this machine's bolt-hold entries, red files the fix task and writes the `adopt-fix` hold. Then `runBeamSteps` brings each subject up (honouring `--no-orch` and a landing-held beam), on green and on red alike.
   A failing check ends the adopt with an error naming the fix task (or, at the limit, the failing tests, the standing task and the hold's re-run), so the command exits non-zero; a check that did not run (its lock held) ends non-zero with the line saying to wait for it.

`--take-hold`: adopt's help names it; the bound is card 2's (only the same subject set moves, no check skipped, no run before a green check).
`--here` keeps `no_subject` and `owned_here`: `hereRequest` drops its adopt-record branch, pulls Bolt once, and ends `owned_here` with no `WriteAdopt` and nothing started when no other machine owns the subject.
`runAdopt` runs `runForeground` and prints its envelope; `callerDeps` keeps `ExemptPause`, `Strand`, `Worktree`, `PullBolt`, `Steps` and `Stderr`; `callFlags` drops `Discard`.
`Command` drops `--keep-local`, `--drop-local`, `--discard` and the `worker` subcommand, adds `--take-hold`, keeps `--archives`, `--no-orch`, `--all` and `--here`, and its long help describes the foreground steps.
`runArchives` refuses `--take-hold` with `--archives` in place of the retired flags.
`outcome` and `slugOutcome` drop the reconcile, mutations, pause, stop, check-start and run-report fields and gain `hold`, `fix_task` and per-subject check states; `adoptFields` renders them, keeping `archives`.
`fixtries.Write` loses its last production callers and goes; `upgradeHub.deps` in dobbyengine's upgrade test seeds `fix-tries.json` through `os.WriteFile` with the file's JSON; fixtries' doc.go names the upgrade's first read as `Read`'s only caller, and `TestReadWrite` (tier 0) seeds the file through `os.WriteFile`.
adoptcli's doc.go is rewritten to the foreground flow, the hold, `--take-hold` and the check's ends.
`cmd/lym`'s `unboundedWaitFiles` drops `internal/adoptcli/follow.go`, and PATTERN-daemon-wakeups.md drops the follow's bullet.
docs/overview.md's adopt entry describes the foreground flow.
PATTERN-dobby-runs.md replaces its "Adopt pause" section with an "Adopt hold" section (an adopt holds the hub through Bolt's holds, written only by boardengine's setters, from its owner write until an adopt check is green; a green check clears it and the bolt-hold entries go with no hand lift), and its `--no-orch` exception names the adopt's last step.
PATTERN.md's overview line becomes, verbatim:

```
- `PATTERN-dobby-runs` — Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt holds the hub through Bolt's holds until an adopt's check is green. — [background](pattern/PATTERN-dobby-runs.md)
```

`TestAdopt` (tier 0) drives `runForeground` over `world`, whose `deps` fill the new seams and record each call; it keeps its refusal and anchor-change rows, the deploy rows (their stop text asserting the hold) and `NestedBeamOwnedElsewhereUnderAdoptedRootIsReportedOwnedElsewhere`, and gains:
`TakeHoldWithAllRefusesNamingBeamStatus`,
`APlainAdoptOfOtherSubjectsUnderThisMachinesHoldRefusesNamingTheHoldsSubjects`,
`AnotherMachinesHoldRefusesNamingItsMachine`,
`TakeHoldRunsEveryStepUnderTheRewrittenHold`,
`AnAdoptOnAMachineWithNoLocalPairAddsThePairBeforeTheTakeover` (call order `AddPair` then `TakeOver`),
`ARunWhosePriorOwnerIsThisMachineIsNotMoved`, `ARunWhoseOriginRecordNamesAnotherMachinesTakeoverIsMoved`, `ARunWhoseOriginRecordNamesThisMachinesTakeoverIsNotMoved`,
`AReRunAfterAKillAtTheOwnerWriteMovesTheSameRunsAndEndsAsAnUninterruptedAdopt`,
`AReRunUnderItsOwnAdoptFixHoldKeepsKindTaskTriesAndFilesNoSecondTask` (the hold's `PriorOwner` is the other machine and the fix task's run has no takeover entry; no `TakeOver` of the fix task's run, and `CatchUp` is called),
`AFailingTakeOverEndsNamingTheRunAndTheHoldsRerun`,
`AHoldTakenOverByAnotherMachineStopsAtTheNextReReadNamingItAndStartsNoDobby` (no `BeamUp` call),
`AHoldClearedMidAdoptStopsAndStartsNothing`,
`AFailingCatchUpEndsBeforeTheDeployNamingTheRerun`, `AFailingWorkspaceIsAWarningAndTheAdoptGoesOn`,
`AGreenCheckBringsEachSubjectUpAndReturnsOk`, `AFailingCheckBringsEachSubjectUpThenExitsNonZeroNamingTheFixTask`,
`StepsPrintInOrder`, `NoOrchLeavesTheOrchDown`.
`TestRunAdopt` (tier 0) keeps its `--here` and orch rows: `owned_here` after one pull with no `WriteAdopt`, `no_subject` outside a subject, and a call from an orch strand passing `--no-orch`; its follow, spawn and discard rows go.
`TestAdoptFields` (tier 0) asserts the new fields. `TestRunArchives` (tier 0) asserts the `--take-hold` refusal.
`TestAdoptIntegration_SecondCloneTakesOver` (tier 1) runs `AdoptAs` from a second hubforge clone sharing only origin; the fixture seeds the hub's gate config with an `adopt` key over a tier holding no test, as `TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers` seeds its tiers, so the check runs green.
It ends with the moved run in the same `runfsm` state as on the first clone, this machine as owner, the takeover entry on origin, the pair added from origin on the clone that had none, and no hold left in Bolt.

**ImpactSummary:** adoptcli's verb, cobra wiring and envelope; cmd/lym's wait list, fixtries' writer with dobbyengine's upgrade-test helper, and the dobby-runs pattern move with it. beamcli and orchcli no longer read adoptcli's or adoptrecord's removed symbols after cards 5 and 6.

**Commit:** 9: Run `lym adopt` in the foreground over pushed state, retiring the worker, the record and the adopt pause
