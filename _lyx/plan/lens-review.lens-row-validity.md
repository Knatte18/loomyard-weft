```yaml
lens: row-validity
outcome: reported
```

Row-validity review of the plan's planned test rows: two blocking findings, five medium, five low.

**BLOCKING**

1. **Card 28: retiring `--window` breaks a test no card fixes.**
   - `internal/battencli#TestBattenRun_RefusesWhileAPauseCoversTheSlug` (`internal/battencli/pausehold_test.go:59-68`) sets `c.windowOpener` and `c.terminalLauncher`, calls `c.runInWindow`, and asserts `windowOpened`.
   - Card 28 deletes `runInWindow`, the window opener and the launcher seams, but does not list this test.
   - Card 26, the earlier card that does list it, only adds an exit-78 assertion and leaves the window parts in place.
   - Card 28's build therefore fails on this test. The fix is to list the test on card 28 and reshape it around `driveRun`.

2. **Card 26: the exit-78 row cannot observe exit 78 the way the test drives batten.**
   - The test calls `c.battenPreRun` and `c.runInWindow` directly (`internal/battencli/pausehold_test.go:66-69`).
   - Card 26 sets the code in `driveRun` through `passExitCode`, which neither call reaches, so the row would pass even with `passExitCode` wrong.
   - The row has to execute the run verb's `driveRun` and read `clihelp`'s recorded code.

**MEDIUM**

3. **Card 15: the respawn-cap test cannot reach the second respawn.**
   - Spawn confirmation is keyed on the process pid (`internal/battenshed/innerrun.go:1118-1125`).
   - The existing row `RunningConfirmedByThisProcessDoesNotSpawn` (`internal/battenshed/innerrun_test.go:603`) shows that a fresh producer in the same test process reads its own marker as confirmed.
   - So `TestInnerRun_RespawnCapStopsWithoutProgress`'s "three passes each respawn" stops after pass 1, and the cap is never reached.
   - The row needs the marker rewritten to a foreign pid between passes, or a pid seam. The card names neither.
   - Card 13's `TestInnerRun_MemoryOutlivesTheProcess`, which also models passes in one process, has the same blind spot wherever it relies on a respawn.

4. **Card 40: the new `TestEndBeforeSessionDown` row cannot see that dobby was spared.**
   - The fake's `endOwned` is `func(string) ([]runproc.Ended, error)` and only records the call (`internal/pairteardown/teardown_test.go:93-95`).
   - Because the sparing happens inside `runproc.EndSparing`, the row passes whether or not `RoleDobby` is spared.
   - The `Teardown.endOwned` seam (`internal/pairteardown/teardown.go:106-107`) has to carry the spared roles, and the fake has to record them. Card 40 lists neither.

5. **Card 40: `TestRegisteredBattenAlive_ReadsADobbyDrivenRun` names no seam for its inputs.**
   - The widened `registeredBattenAlive` needs batten's status, the run's parent and the cancel marker.
   - The card says "runproc records ... under a temp dir" but names no seam for those three. Today's `Teardown` reads batten's state through `battenRun`, and the parent through `hubgeom.ParentOf` over a real prime.
   - The halted-reads-not-alive and cancel-marker rows therefore cannot be driven without seams or files the card does not state.

6. **Card 18: the `TestBattenDriverDeps` routing row is out of reach.**
   - `TestBattenDriverDeps` is a table of nil and non-nil seam checks (`internal/battencli/driverdeps_test.go:35-57`).
   - The hand-back `Notify` posts through the real `rowNotices` and creel, so "routes a Land stop to `RowStopNotice`" cannot be observed from that table.
   - The routing needs an injectable notify, or a pure helper choosing between `RowStopNotice` and `HandBackNotice` with its own rows.

7. **Card 14: a spawn-hold row can stand without its intended input.**
   - The row "a covering pause lifted before the next Call spawns on that Call" needs `Pause` to answer covered and then uncovered.
   - `newInnerRunDeps` has a `Pause` seam, but this lifted case is already the absent-status spawn path that `TestInnerRun_SpawnsUntilASpawnIsConfirmed` covers.
   - The row only adds value if it asserts the first Call made no spawn. As written it is a near-repeat.

**LOW**

8. **Card 29: one row is planned twice.** "A later resume that reaches origin and exits 0 removes the marker" appears in both `TestOriginOwedMarker` and the new `TestRunCmd_ResumeDispositions` row. Keep one.

9. **Card 37: `TestDobbyCmd_ExitsOnAHeldLock` repeats card 32 and may not be tier0.**
   - The held-lock exit is already pinned by `TestRun_LockOwnerAndWakeups`'s held-lock row in card 32.
   - `dobbyCmd` resolves the prime from the worktree root through `lyxcwd`, and `ResolveWorktree` goes through `gitWorktreeRoot` (`internal/lyxcwd/lyxcwd.go:118`). So the claimed "tier0, temp geometry" may not hold.

10. **Card 13: `TestInnerRun_MemoryOutlivesTheProcess`'s "state-changed notice goes once across three passes" row repeats an existing test.** `TestNotice_RestartDoesNotRenotify` already covers it through the notice-episode marker file. The new row should pin only what the memory record adds: delivery attempts and spacing, the idle lock, the failure ticks.

11. **Cards 38 and 39: two new top-level tests could be rows of existing ones.**
   - `TestStart_StartsTheDobbiesFirst` (card 38) runs on the existing start harness (`internal/orchcli/start_test.go:154`), as the `TestStart_*` family does.
   - `TestAdoptRefusal` (card 39) runs on `TestAdopt`'s `newWorld`.
   - Both could be rows of the existing tests; the rubric's "Test economy" asks for that.

12. **Card 24: the loom-reader rows in `TestEligibility_BattenRule` partly repeat `TestCheck_RefusesLandingHalts`.** The landing-halt answer under loom readers is already pinned there, so those rows should be limited to the statuses that test does not cover.

Tool calls:
1. `internal/pairteardown/teardown_test.go` fakes and `sessiondown_test.go`: my context lacked the `writeClaim` and `endOwned` fake shapes.
2. The batten pause test locator, the `TestBattenDriverDeps` table, and spawn-confirmation use in `innerrun_test.go`: my context lacked these test bodies.
3. The `pausehold_test.go` head, the README golden form, and `ResolveWorktree`: needed to see what the pause test calls.
4. `pausehold_test.go:53-90`, the README golden source and `TestEnd`'s table shape: needed the pause test's assertions.
5. `ResolveWorktree`'s git use and the orch start harness: needed for the tier claims of cards 37 and 38.
