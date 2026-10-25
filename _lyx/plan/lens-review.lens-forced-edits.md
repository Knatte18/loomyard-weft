```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: what each card's change breaks outside the members it names.

**BLOCKING**
- **Card 28 (batten-first-pass), deleting `internal/battencli#battenCLI.runInWindow`.** `internal/battencli/pausehold_test.go:68` calls `c.runInWindow(cmd)` inside `internal/battencli#TestBattenRun_RefusesWhileAPauseCoversTheSlug`. Only card 26 lists that test, and card 26 runs before the delete, so the package stops compiling at card 28. Card 28 must list `internal/battencli#TestBattenRun_RefusesWhileAPauseCoversTheSlug` and drop its window arm.
- **Card 28, retiring `--window`.** `internal/battencli/startguard_integration_test.go:91` runs `RunCLIIn(..., []string{"run", slug, "--window"})` inside `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite`. With the flag gone, cobra refuses an unknown flag, so the test asserts the wrong thing. No card lists it.
- **Cards 5 and 7 (draft rule and start-guard refusal).** Integration fixtures create tasks with `UpsertTask` and then start them with a fresh `lyx batten run`. Every such task is now `draft`, which the new guard refuses. Unlisted sites:
  - `seedBoardTask` (`internal/battencli/lifecycle_integration_test.go:203`)
  - `seedBeamBoard` (`internal/battencli/beamfork_integration_test.go:42`)
  - `internal/battencli#TestBeamLanding_ReachesWorktreeCreateSkip` (`beamland_integration_test.go:71`)
  - `internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver` (`adopt_integration_test.go:64`)
  
  Each needs a release (`SetStatus` nil) after the create, or the batch and webster gates fail. `internal/loomcli#TestRunOwnerVerdict` (`ownercheck_integration_test.go:72`) upserts the same way and should be checked too.

**MEDIUM**
- **Card 4 (dobby-records), the `beam.yaml` loader.** It uses `configengine.LoadOrTemplate`, so `internal/dobbyengine` must join `configStrictnessDegradingSet` in `cmd/lyx/configstrictness_test.go:47`, a pinned set. No card lists `cmd/lyx#configStrictnessDegradingSet`.
- **Card 13 (innerrun-short-check) lists only part of the test set.** Deleting `wait` turns every multi-check Call into one check plus `Yield`. The Intent says "the InnerRun and notice tests that drive a wait inside one Call … drive successive Calls instead", but many of those tests are not listed:
  - every `TestInnerRun_Awaiting*` test
  - `TestInnerRun_DoneLivenessErrorIsTreatedAsLive`, `TestInnerRun_DoneFinishesPastGraceWithLiveDriverAndWarns`, `TestInnerRun_UnparseableDoneSeenMarkerRestartsTheGrace`, `TestInnerRun_StaleDoneSeenMarkerIsClearedWhileRunning`
  - `TestInnerRun_HaltWarnsOncePerEpisode`, `TestInnerRun_ReHaltAfterObservedResumeWarnsAgainAtTheSameHistoryLength`, `TestInnerRun_PausedThenRunningThenDoneReachesDone`, `TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn`, `TestInnerRun_ARunningChildEndsTheReviveEpisode`
  - `TestInnerRun_BinaryChange*` with `newRestepHarness`, and `newLockHarness`
  - `fakeClock.Sleep`
  - `TestInnerRun_SpawnsUntilASpawnIsConfirmed`, `TestInnerRun_AdoptsARunningChildWithADriver`, `TestInnerRun_FailedSpawnIsRetriedOnTheNextCall`, `TestInnerRun_RunningArmReviewWaitNote`, `TestInnerRun_NilSeamsDefaultToStdlib`, `TestInnerRun_CancelledContext`
  - every `TestNotice_*` test except `RestartDoesNotRenotify`, and `TestNotice_EligibleStopHold`
  - the `stophold_test.go` and `approvalacted_test.go` tests
  
  The card either names them all or states which keep passing unchanged.
- **Card 13, docs.** `docs/overview.md`'s batten paragraph still says "`Run-Shed` waits on its child inside its own call rather than bouncing once per poll … A running child is waited on without a time limit". No card edits those sentences: card 28 edits only the `--window` ones.
- **Card 13 or 32, the daemon-wakeups background.** `pattern/PATTERN-daemon-wakeups.md` still lists "batten's Run-Shed wait" as an unbounded wait and does not list dobby's loop. Card 32 adds `internal/dobbyengine/loop.go` to `cmd/lyx#unboundedWaitFiles` but no card updates the background file, and `internal/battenshed/innerrun.go` stays on the list with no wait left in it.
- **Card 21 (arbiter-key-copy), the new `Result.Copied` field.** The Intent says "logged by the walk", but the walk's log line is `internal/hubreconcile/walk.go:383`, which no card lists. `lyx config reconcile`'s result rendering, `internal/configcli/configcli.go:610-617`, would not report `Copied` either.
- **Card 40 (teardown-dobby-liveness).** `EndBeforeSessionDown` ends processes through the `t.endOwned` seam, which is wired to `internal/pairteardown#Teardown.endOwnedProcesses` and calls `runproc.End`. Sparing `RoleDobby` needs that member, or a new seam in `pairteardown`'s `New`, edited too. Without a seam, `TestEndBeforeSessionDown`'s new row has no way to observe the spare over its fakes.
- **Card 41 (finalize-key-notice), `ConfigChanges.Contents`.** The tier1 `internal/fabricengine#TestReadConfigChanges` (`configchanges_integration_test.go:24`) asserts what `ReadConfigChanges` returns and is not listed; the new `Contents` field needs a row there.

**LOW**
- **Cards 5 and 6 (draft and release).** `plugins/ly/skills/board/SKILL.md:37` documents `set-status … "status":null` as a plain clear. Neither it nor `lyx board set-status`'s help (`internal/boardcli/cli.go:627`, "null clears") says that null on a task is a release and that a new task is `draft`; no card lists either.
- **Card 18 (row-notices-after-chain).** Once Teardown's failure notice goes, `TeardownDeps.Notices` (`internal/battenshed/deps.go:403`) and its fill in `battencli#battenCLI.wire` are left unused. The card neither removes them nor states why they stay.
- **Cards 4, 22 and 32 (new packages).** `internal/dobbyengine` and `internal/shedarbiter` claim told geometry, but no card creates the `seam_enforcement_test.go` import-allowlist test their siblings carry (`internal/sheddriver/seam_enforcement_test.go`, `internal/battenshed/seam_enforcement_test.go`).

Tool calls:
1. Grep of arbiter config readers, `postNotice` and `ender.end` callers, the exit-sweep guard and the geometry lists: needed every caller of the re-signed and deleted members.
2. `cmd/lyx/configstrictness_test.go`, plus a grep for tests that start batten runs or assert notice text: needed the pinned loader set and the tests the draft rule and notice removal affect.
3. Grep of the battencli notice, pause-hold, start-guard and window-site tests: needed which tests use the retired window path.
4. Grep of task creation in integration tests, the configreg names pin, and the walk and configcli result logging: needed the fixtures that create tasks and the readers of `Result`.
5. The functions enclosing the `UpsertTask` lines, the `ConfigChanges` tests, the plugin skill text and the `batten:` prefix users: needed exact glyphs and the remaining doc and test sites.
