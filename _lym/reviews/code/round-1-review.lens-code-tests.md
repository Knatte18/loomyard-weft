```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list**

1. `darnengine.IsBenchPairSlug` — the `<slug>-b<N>` form, zero, leading zero, shared prefix, other task: pinned by `TestIsBenchPairSlug` (`internal/darnengine/bench_test.go:86-113`).
2. `darnengine.IsWriter` (plain/numbered darn role, non-darn role): pinned through `TestIsMergeTaskWriter` rows `NumberedRole` and `OtherRoleOfAMergeTask` (`internal/darnengine/mergetask_test.go:23-24`), plus the guard subtest's `NumberedWriter`/`NotADarnWriter` rows (`roleguard_test.go` diff lines 412, 421).
3. `IsMergeTaskWriter` refactor: pinned by `TestIsMergeTaskWriter`.
4. `benchPairVerb` admission — add/remove from the worktree and the records sibling: `AddFromTaskWorktree`, `RemoveFromTaskWorktree`, `AddFromRecordsSibling`.
5. `benchPairVerb` refusals — `--parent` (`AddWithParent`), `--adopt-local` (`AddAdoptingLocal`), slug outside the form (`SlugNotABenchPair`), another task's slug (`BenchPairOfAnotherTask`), another worktree (`FromAnotherWorktree`), no record (`NoRunRecord`), other profile (`RunOnAnotherProfile`), other recipe (`RunOfAnotherRecipe`), other role (`NotADarnWriter`): pinned.
6. `benchPairVerb` with more than one positional argument (`len(args) != 1`): none.
7. `benchPairVerb` on a `run.json` that exists but does not decode (`err != nil` refusal): none.
8. `benchPairWayForward` appended only for a darn writer's refused add/remove: pinned by the way-forward assertion in the loop (`roleguard_test.go` diff line 435) across the writer's refused add/remove rows and the webster negative; no row checks that a darn writer's refusal of some other verb lacks the clause (minor).
9. `injectedPathsAreOwn` admitting a bench pair: `WritersBenchPair`, `BenchPairInAnotherHub`, `BenchSlugUnderAnotherRole` (diff lines 450-452).
10. `pairLocation`/`strandHubWorktree` refactor of `isOwnWorktree`/`isOwnPairPath`: covered by the guard test's existing own-pair and push rows (outside this diff's added rows).
11. `shedrun.ReadRunUsageRecord` — found / not found / decode error: reached only through the guard rows; not-found pinned by `NoRunRecord`, found by the admission rows; decode error and non-NotExist read error: none.
12. `RemoveBenchPairs`:
    - removes and archives every bench pair, uncommitted file at the archive tag: `EveryBenchPairIsArchivedAndRemoved`;
    - leaves the shared-prefix pair: `requirePairStanding(sharedPrefix)` in every row;
    - ends a live process rather than refusing: `ALiveProcessOfABenchPairIsEndedNotRefused`;
    - skips a pair a board entry names: `ABoardEntryNamingABenchSlugKeepsThatPair`;
    - failed archive returns the error and keeps the pair: `ARefusedArchiveTagPushKeepsTheBenchPair`;
    - skips a pair a batten run directory names: no row in `TestRemoveBenchPairs`; the shared `runNamingReason` reason is pinned via `TestRemoveOrphanPair`'s `ABattenRunDirectoryNamingThePairRefusesTheRemoval` (`leftover_integration_test.go:112-118`), which pins the reason, not the bench path's skip-and-continue;
    - returns the slugs removed before a failure: none (the error row returns early without checking `removed`);
    - a failed `td.Run` removal (as opposed to archive): none.
13. `requireStillOrphan` rewritten over `runNamingReason`: pinned by `TestRemoveOrphanPair`'s existing rows on the error text ("a batten run directory names it").
14. Worktree-Teardown `Remove` wiring in `internal/battencli/wire.go` — bench removal runs before the task pair's removal, a bench failure halts the row with the task pair standing, the error's retry text: none. `TestWorktreeTeardown_RemoveFailureNamesRemovalHalf` (`internal/battenshed/teardown_test.go:150`) pins only the producer's handling of a generic Remove error; no battencli test drives the wired closure or `RemoveBenchPairs`.
15. `runprofile.DocsProfile` / `retroengine.TaskProfile = DocsProfile`: pinned by `TestTemplateDocsProfile` (`internal/runprofile/runprofile_test.go:230`), `TestPlan_TaskFilesOneDarnTaskOnTheDocsProfileWithAnOpusWriter` (`internal/retroengine/retro_test.go:188`) and `internal/retrocli/fileretro_integration_test.go:205`.
16. Doc comments / `doc.go` prose: exempt.

**Findings**

- **MEDIUM** · class `design` · `internal/battencli/wire.go:1242-1251` (`Teardown.Remove` closure) — the behaviour the diff adds here (bench pairs removed before the task pair; a bench failure halts the row with the task pair still standing) has no test; a reordering or a dropped `return` on error passes every test. Evidence: no `*_test.go` in `internal/battencli` or `internal/loomcli` mentions `RemoveBenchPairs` or bench pairs; the battenshed producer test (`internal/battenshed/teardown_test.go:150`) drives only a fake Remove.
- **LOW** · class `design` · `internal/pairteardown/leftover_integration_test.go` (`TestRemoveBenchPairs`) — two of `RemoveBenchPairs`' contract clauses are unpinned: "the first failure returns with the slugs removed before it" and a failed pair removal through `td.Run` (as distinct from a failed archive); the error row returns before asserting `removed`. Evidence: the early return after `requirePairStanding(t, h, bench)` (diff lines 688-693); doc comment at `leftover.go` diff line 517.
- **LOW** · class `design` · `internal/fabriccli/roleguard_test.go` (bench subtest) — two refusals in `benchPairVerb` have no row: a `run.json` that does not decode (`err == nil &&`), and an invocation with more than one positional argument (`len(args) != 1`); removing either check passes the tests. Evidence: `roleguard.go` diff lines 276-277 and 296; subtest rows diff lines 409-421 only ever write a decodable record with one slug argument.

Tool calls:
1. Grep of test files for `ReadRunUsageRecord`/`IsWriter`/`IsMergeTaskWriter`/`DocsProfile` tests and the Remove wiring — the manifest held no test files beyond the diff's own.
2. Read of `TestIsMergeTaskWriter` rows and the `wire_seams_test.go` seam block plus grep for bench tests in battencli/loomcli — to settle whether `IsWriter` and the Remove wiring are covered.
