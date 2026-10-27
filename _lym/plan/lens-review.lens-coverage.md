```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and card branch for a planned test row that would fail if it broke.

**MEDIUM — Decision 5 / criterion "`dobby.out` past the bound at a start of the lock holder rotates the same way where `selfexec` execs":** no row tests this rotation.
- Card 10, step 4 of `runDobbyWith` runs `dobbyengine.RotatePastBound(geom.OutPath)` and `reopenStdio` once the lock is held, but no test reaches it.
- `TestAppendLog` covers `AppendLog` only. `TestRotateOutBeforeSpawn` covers only the Windows spawner path.
- `TestDobby_ReexecsOntoAReplacedBinary` never takes `dobby.out` past 1 MiB.
- Evidence: `_lym/plan/10-dobby-cli-exits-and-out.md` step 4 and its Tests list; `_lym/plan/06-dobby-log-file.md` Tests.

**MEDIUM — Decision 11 / criterion "a running record whose pid is alive but whose start ticks differ shows ended without a verdict":** no row tests the start-ticks comparison.
- `TestCheckView` passes a ready-made `alive` bool into `checkView`.
- `TestEnsureCheck` drives "differing ticks through the `alive` seam".
- So `checkAlive`'s start-ticks comparison, and its no-ticks-reads-alive default, are never tested, though `ReadCheckView` and `ensureCheck` both rely on it.
- Evidence: `_lym/plan/15-beam-status-adopt.md` Tests; `_lym/plan/14-adopt-spawns-check.md` (`plan:internal/adoptcli#checkAlive`, `TestEnsureCheck`).

**MEDIUM — Decision 1, the kept `State.Tidy`:** no row tests the replacement step.
- The record says the kept value is replaced, nil included, by the last tick that ended with the marker present.
- Every row of `TestRun_WorktreeGoneExits` removes the worktree on tick 1, so the kept value is always the one `recordStart` read.
- No row settles a tidy on a tick with the marker present, removes the worktree later, and expects no `StopStrand`.
- Evidence: `_lym/plan/08-dobby-worktree-gone.md`, "The kept tidy" paragraph and its rows.

**MEDIUM — Decision 1 / criterion "stops a tidy agent … through a runner over the prime":** no test pins the runner's wiring.
- Card 8 has `hubDobbyDeps` set `PrimeTidyShuttle` to a `runnerTidyShuttle` over the prime, for a beam dobby only.
- No row checks that it is set for a beam, unset for the prime, or built over the prime rather than the worktree. The tier0 row uses a fake.
- Evidence: `_lym/plan/08-dobby-worktree-gone.md` Edit `internal/beamcli#hubDobbyDeps` and its Tests.

**MEDIUM — Card 8, harness `Log` conflicts with the file assertions:** the rows cannot assert what they claim as written.
- Card 7 makes `newLoopHarness` fill a recording `Log` fake.
- Card 8's rows assert that "the prime log holds the exit line" and that nothing is created at the removed path. That needs `Deps.Log` to be `dobbyengine.AppendLog`, or the fake to record the path.
- Neither card states which. Over the recording fake, the "creates nothing" half proves nothing about `AppendLog`'s no-mkdir rule.
- Evidence: `_lym/plan/07-dobby-start-and-exits.md` Tests (harness); `_lym/plan/08-dobby-worktree-gone.md` rows.

**MEDIUM — Decision 6, heartbeat operand:** the heartbeat sits in a step that a tick can skip.
- The record says the heartbeat is written "by the first tick ten or more minutes after the last heartbeat".
- Card 9 writes it in `drivePasses`, which `tick` skips while an anchor-move record stands (`internal/dobbyengine/loop.go:302-309`), and which returns early when `Runs` fails.
- So a live dobby writes no heartbeat during a standing move or a failing runs read, and looks dead. No row covers either case.
- Evidence: `_lym/plan/09-dobby-change-log-heartbeat.md` "Heartbeat" paragraph.

**MEDIUM — Decision 3 / criterion "Each exit path of Decision 3 writes its line":** the fresh-start line is never asserted.
- Card 7 logs `start fresh pid <pid>`, but no row of `TestRun_LockOwnerAndWakeups` or `TestRun_Heartbeat` asserts it.
- Only `start re-exec` is asserted, in the tier1 test.
- Evidence: `_lym/plan/07-dobby-start-and-exits.md` Run step 4 and its Tests.

**MEDIUM — Criterion "a refusal with no resolved worktree, in a process this binary spawned, leaves its envelope in `dobby.out` and nothing in `dobby.log`":** only the second half is planned.
- The loose-directory row of `TestRunDobbyWith_TracesEachExitBeforeRun` asserts only no notice and no `dobby.log`.
- The `dobby.out` half has no row.
- Evidence: `_lym/plan/10-dobby-cli-exits-and-out.md` Tests, first bullet.

**LOW — Criterion "A second check started while one runs exits 0, writes its envelope to `check.out`":** the exit code and envelope are untested.
- `TestRunCheck` covers only `runCheck` reporting that it did not run.
- `checkCmd`'s `already_running: true` envelope and exit 0 are never asserted. Neither is the spawn's stdout redirection to `check.out`, which `startSuite` refuses under test.
- Evidence: `_lym/plan/13-adopt-check-verb.md` `checkCmd` and its Tests; `_lym/plan/14-adopt-spawns-check.md` `startSuite`.

**LOW — Card 13, a gate that ends with no tier:** no row covers it.
- `runCheck` step 5 ends `failed` "with `FailedTier` the first failed tier".
- No row covers a `RunGate` that returns non-zero before any tier ends, such as a gate-key refusal or slots busy. There is then no failing tier for the end notice to name.
- Evidence: `_lym/plan/13-adopt-check-verb.md` steps 4–5 and its Tests.

**LOW — Decision 10, "The verb resolves the prime from the hub whatever its cwd":** no row runs `checkCmd` from a beam or a subdirectory.
- Evidence: `_lym/plan/13-adopt-check-verb.md` `checkCmd`.

**LOW — Decision 3, wiring failure whose `ResolveWorktree` fails:** no row covers it. Card 10 step 3's "the line is the only trace" branch is unreached.
- Evidence: `_lym/plan/10-dobby-cli-exits-and-out.md` step 3 and its Tests.

**LOW — Card 7 step 1, a failed `ReadsAsReexec` read:** no row covers the branch that keeps its line only.
- Evidence: `_lym/plan/07-dobby-start-and-exits.md` Run step 1.

**LOW — Card 3, a failed `machinename.Name` read in the `Landings` seam:** no row covers the error return through the tick-failure episode.
- Evidence: `_lym/plan/03-landings-by-machine.md` `hubDobbyDeps` paragraph.

**LOW — Decision 3, the post-re-exec refusal notice's content:** it is not asserted.
- The record says it names the refusal's error and its way forward.
- Card 10's rows assert only "one notice". Card 7's row for a `Locked` error does assert the stop line and the notice.
- Evidence: `_lym/plan/10-dobby-cli-exits-and-out.md` Tests.

I made no further tool calls.
