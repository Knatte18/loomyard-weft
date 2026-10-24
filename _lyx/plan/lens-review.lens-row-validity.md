```yaml
lens: row-validity
outcome: reported
```

Findings (row-validity):

1. MEDIUM — Card 12, the `TestListTasksBriefLayerAndProposal` row "parent_beam from a ParentOf fake" can't be reached. The test is in the external package (`boardengine_test`) and builds a bare `boardengine.NewStore("")` (internal/boardengine/store_test.go:758-759). `Store.parentOf` is unexported and only `Board` sets it (internal/boardengine/board.go:66, `store.parentOf = b.parentOf`), so the test has no way to inject the fake. Fix: make the row a `Board`-level row built with `Config.ParentOf`, or put it in a new test that goes through `Board.ListTasksBrief`.

2. MEDIUM — Card 9, the `TestListTasksBriefLayerAndProposal` row "carries the project's … owner" can't fail if the code is wrong. A `NewStore("")` store has no prime owner, and `Owners.Effective` returns "" whenever `o.Prime == ""` (internal/boardengine/owners.go `Owners.Effective`). So the owner cell reads "" whether or not a project's owner is resolved. The row needs a board whose `board.json` records `prime_owner`, through a board dir or a `Board`.

3. MEDIUM — Card 21, the watcher rows: the beam watcher's hub-pause row "settles on a beam orch's own strand" needs a strand name the `Watcher` does not hold. Today both checks match an orch by `name.Slug == ""` (internal/orchengine/pause.go:149, :190). The `Watcher` struct keeps only the cursor's strand GUID (internal/orchengine/watcher.go:128), and `State.Strand` is a GUID, not a name. The card lists no new watcher field, no zero-value meaning for it, and no orchcli site (`watchCmd` / `spawnWatcherProcess`) that fills it. So the row in `TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine` (internal/orchengine/watcher_pause_test.go:121) has no input that reaches a beam-strand comparison.

4. MEDIUM — Card 35, the `TestCancelRun` row "the two promote halves together do what a cancel without --remove does" can't be driven over fakes. `TestCancelRun` drives `cancelRun` with a fake `cancelDeps` (internal/battencli/cancel.go:29). The card creates only the hub-level exports `EndRunForPromote(ctx, prime, slug)` and `ClearRunForPromote(ctx, prime, slug)`, which take a location and build real deps. The halves that take `cancelDeps`, the ones `cancelRun` "splits into", are not declared as `Create` handles, so the row has nothing to call. Declare the two deps-taking halves.

5. MEDIUM — Card 26: `LandDeps.ReadyRoutes` has no stated meaning when nil. The existing `TestLand_ReadyNotice` (internal/battenshed/land_test.go) and every other Land test build `LandDeps` without the new seam. The card does not say whether a nil `ReadyRoutes` keeps today's "all newly ready slugs ride the landing line" or names none. If it names none, the existing ready-notice rows fail.

6. LOW — Card 9 contradicts itself. The `TestSetOwners` row says it "refuses a claim on a project whose beam is false", while the Intent says `writeOwners` writes a claim only on a task or a beam project, "leaving any other project's status alone", which is a skip, not a refusal. Make the row assert that no claim is written and that `Recorded`/`Lost` report it, or make the Intent refuse.

7. LOW — Cards 9 and 10: `TestProjectRules` drives a `Board` "over a temp board directory with a configured vocabulary" with a `PairExists` fake. `boardCriticalSection` refuses a `Board` with no `Readme` output (internal/boardengine/board.go `boardCriticalSection`, first check). No builder is declared or used: the existing ones are per-file (`newRenameBoard` internal/boardengine/rename_test.go:25, `newIntakeBoard` internal/boardengine/intake_test.go:21). Card 10's `TestRename_Refusals` row "an entry with a pair" also needs `Config.PairExists` on `newRenameBoard`, which card 10 does not list as an Edit target.

8. LOW — Card 29: the `TestFinalize_MergeJudgement` rows "each run the gate and land" overclaim. That table asserts only reroute versus proceed (`wantReroute`, "with no gate run", internal/landingshed/finalize_merge_test.go:57-58, and the row "an up-to-date merge proceeds to the gate", :72). It never runs the gate or lands. Reword the rows to "proceed to the gate with no reroute and no overlap note".

9. LOW — Card 28, test economy: `TestWorktreeCreate_ForksFromTheBeam` is a new top-level test of the `CreateWorktree` closure. `TestWire_CreateWorktreeClaimsTheBoardEntry` (internal/battencli/wire_seams_test.go:410) already drives that closure, so the beam fork belongs as rows there. The new `startGuard` `ParentBeam` seam also has no stated nil meaning; the guard's convention is that a nil seam checks nothing, which would skip the dirt check for every task.

10. LOW — Card 22: the `TestOverviewSource_DrawsTheOrchLine` row "an unreadable beam state broken on its own line only" needs the orch list injectable. The card has `newOverviewSource` re-read `orchcli.HubOrchs` on each frame with no seam on `overviewSource`, so the test cannot supply a fake beam orch path list.

11. LOW — Card 32: the `TestWire_BeamRecipe` assertion "a Finalize built with `LargeMergeLands`" reads a `landingshed.Deps` field inside a built shed producer, which `*shedengine.Shed` does not expose. Assert on the `landingDeps` result for a beam recipe instead, as `codeReviewOff`/`fixerPlanDir` are tested.

Tool calls:
- grep of `internal/battencli/cleanup_test.go`, `internal/boardengine/store_test.go` and several test files: to see how the cleanup skip is compared, how `TestListTasksBriefLayerAndProposal` builds its store, and whether the named tests exist; my context had only the function names.
- sed of `internal/orchengine/watcher.go` and grep of `internal/landingshed/finalize_merge_test.go`: the watcher's fields and the `MergeJudgement` table columns, which my context lacked.
- grep of `internal/orchengine` for the watcher's strand-name source and the hub-pause test: to confirm no own-name field exists.
- grep of the `internal/boardengine` test helpers, `TestUpsertFieldAllowlist` and `TestOwnersEffective`: to check which `Board` builders and fakes exist.
