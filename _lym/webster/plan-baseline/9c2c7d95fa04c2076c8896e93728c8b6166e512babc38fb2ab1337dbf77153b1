# Card 21 — dobby-catch-up

**Create:**
- `internal/dobbyengine#CatchUpWait`
- `internal/dobbyengine#daemon.catchUp`
- `internal/dobbyengine#daemon.syncBoltHold`
- `internal/dobbyengine#TestRun_CatchUp`
- `internal/dobbyengine#TestRun_SyncsBoltHoldEntries`

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#daemon`
- `internal/dobbyengine#Run`
- `internal/dobbyengine#daemon.recordStart`
- `internal/dobbyengine#daemon.tick`
- `internal/dobbyengine#daemon.handOver`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#daemon.passExited`
- `internal/dobbyengine#State`
- `internal/dobbyengine#View`
- `internal/dobbyengine#BuildView`
- `internal/dobbyengine#newLoopHarness`
- `internal/dobbyengine#TestRun_ForeignOwner`
- `internal/dobbyengine#TestDrivePasses_ExitHandling`
- `internal/dobbyengine#TestBuildView`
- `internal/beamcli#hubDobbyDeps`
- `internal/dobbyengine/doc.go`

**Delete:**
- `internal/dobbyengine#daemon.pauseRunsForOwner`

**Uses:**
- `internal/dobbyengine#CatchUp`
- `internal/battencli#HubCatchUp`
- `internal/battencli#DobbyPairSlugs`
- `internal/tenderengine#SyncBoltHold`
- `internal/tendercli#HubDeps`
- `internal/pairteardown#EndSubjectProcesses`
- `internal/parentlock#Store.HeldBy`

**Intent:** Run the catch-up in the dobby after every absence it sees, before any act rule answer, and keep its runs' `bolt-hold` entries in step with the Bolt hold (D4, D5).

`Deps` gains `CatchUp CatchUpDeps`, `SyncBoltHold func(ctx context.Context, scope, held []string) error` and `EndSubject func() error`.
A `CatchUp` whose `PullBolt` is nil means no catch-up, as a nil `Deps.PullBolt` pulls nothing, so `newLoopHarness` and every hand-built `Deps` keep running passes; a nil `SyncBoltHold` syncs nothing and a nil `EndSubject` ends nothing.
The daemon gains `catchUpOwed`, `catchUpNext`, `catchUpDelay` and `handedOver`, and one notice episode per failure class.

The catch-up is owed:
at a fresh start (`recordStart`'s non-re-exec path; a re-exec is no absence);
when the tick's sleep watch sees a sleep;
when a pass exits `PassExitOriginWait` or `PassExitOwnedElsewhere`;
and when `drivePasses` lists a run whose verdict answers `hand_over` and whose `RunView.PairPresent` is true, so a run another machine took while its pass was in flight here (InnerRun holding on its owner verdict, or loom's pre-push stop) is handed over and its processes end.
`DobbyRuns` lists every batten record of the worktree whatever pair stands here, so a run whose pair is not here, one already handed over or one whose pair never stood here, owes nothing.
`PairPresent` is a path check on the pair's worktree, while the hand-over removes a pair only when it reads as a registered linked worktree, so a directory left standing with no registration survives a successful catch-up.
A run that owed the catch-up through this trigger and that a successful catch-up then covered owes none from it again while it still lists as `hand_over`:
the daemon keeps its slug in `handedOver`, a map `Run` creates beside the other per-run maps and that the catch-up's success does not drop, and forgets it once the run stops answering `hand_over` or is no longer listed.
Card 20's `DobbyPairSlugs` reads "whose pair stands here" by the same predicate as `PairPresent`, so the trigger's set and the catch-up's scope agree.
`tick` runs `catchUp` first, after the sleep watch and before any scheduling:
while it is owed and its next try is due, it runs `CatchUp(d.deps.CatchUp)`.
Success clears it, resets the delay, clears the state record's wait and owes every listed run a full pass, dropping the per-run memory (`childSeen`, `liftPending`, `wasCovered`) of the runs it touched; each run's next pass is a full pass.
Failure keeps it owed and schedules the next try a delay later, doubling from one second to `idleCeiling` (`PATTERN-daemon-wakeups`: the loop's own one-second floor bounds the wake), and records `State.CatchUp` as a `CatchUpWait` with a reason, the detail and the time:
`origin` for `ErrOriginUnreachable`;
`refused` for a registration refusal, whose notice names the refusal's own way forward (for an unresolved config directory: set the variable it names, then `lym beam stop` and `lym beam start`, since the dobby keeps the environment it started with);
`failed` for every other error (a Bolt pull skipped as diverged or dirty, a run's push or hand-over that failed, a failed first read), whose notice names the detail and the way forward: fix the cause the detail names; the dobby retries on its own.
A `refused` or `failed` wait posts one notice per episode; `origin` posts none, since it clears when origin answers.
While the catch-up is owed the tick spawns no pass, starts no task, takes no landing step and runs no tidy step; the state record's upkeep and the landing count go on, as under an anchor change.
`View` gains `catch_up` from the record, so `lym beam status` shows the dobby waiting on origin, refused or failed.

After the catch-up and before the passes, `syncBoltHold` hands `SyncBoltHold` the scope and the held list:
the scope is every slug `DobbyPairSlugs` gives for this dobby, not only the runs the tick listed, so a run `DobbyRuns` left out of the tick still has its entry dropped once the hold clears;
the held list is the listed runs whose verdict reason is `held`, so a run whose push is owed (reason `push_owed`) gets no entry and its push passes the gate;
a held run holding its parent lock gets no entry either, through `SyncBoltHold`'s parent-lock read (card 15).
A hold read at a Bolt pull applies at once; a failure warns once per episode.

`handOver`, the beam's hand-over when the beam reads owned elsewhere, runs the catch-up in place of `pauseRunsForOwner` (deleted), so its runs are handed over (processes ended, tips archived, pairs removed) instead of paused.
Once the catch-up succeeds it calls `EndSubject`, which ends the subject's processes with the dobby end set (D4: "The dobby's own handover of a subject calls `pairteardown.EndSubjectProcesses` with the dobby end set"); the calling dobby is never ended by it and exits on its own.
A failed catch-up keeps the dobby up and retries at the next probe tick like a failed archive.
The periodic Bolt pull stays.

`hubDobbyDeps` wires `CatchUp` to `battencli.HubCatchUp(prime, DobbyPairSlugs(...))`, `SyncBoltHold` to `tenderengine.SyncBoltHold` over `tendercli.HubDeps`, with the machine and standing read from the local Bolt's holds and `HoldsParentLock` reading `parentlock.Store.HeldBy` over the Bolt's parent lock directory (`parentlock.Dir`), and `EndSubject` to `pairteardown.EndSubjectProcesses(prime, owner, true)` with the beam's slug, or `runproc.HubOwner` for the prime.

Tests, tier0:
`TestRun_CatchUp` over `newLoopHarness` covers:
a fresh start runs the catch-up before the first pass and a re-exec runs none;
a sleep the watch sees runs it before the next pass;
an unreachable origin spawns no pass, records the `origin` wait and retries after the delay;
a refusal records `refused`, posts one notice and spawns nothing;
any other failure records `failed` with its detail and posts one notice;
a later success clears the wait and gives full passes;
a listed run whose verdict answers `hand_over` owes the catch-up, gets no pass and is handed over;
a listed run answering `hand_over` with `PairPresent` false owes no catch-up and the next tick runs none;
a listed `hand_over` run whose pair stays present after a successful catch-up owes the catch-up once, not on each tick;
a zero `CatchUp` runs passes as before.
`TestRun_SyncsBoltHoldEntries` covers the scope from `DobbyPairSlugs` and the held list handed over, a run reading `push_owed` under a hold left out of the held list, an empty held list once no run reads held, and a failed sync logged once while the tick goes on.
`TestRun_ForeignOwner` asserts the hand-over runs the catch-up instead of pausing runs and then calls `EndSubject`, and that a failed catch-up keeps the dobby up and retries at the next probe tick.
`TestDrivePasses_ExitHandling` gains `PassExitOriginWait` and `PassExitOwnedElsewhere` owing the catch-up.
`TestBuildView` gains the `catch_up` field.
`doc.go` describes when the dobby catches up and what it holds meanwhile.

**ImpactSummary:** Deps gains three seams with one production filler, the tick gains a gate ahead of scheduling, and the view gains one field; the beam hand-over swaps its owner pause for the catch-up.
