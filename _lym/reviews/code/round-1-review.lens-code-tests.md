```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, covering each behaviour the diff adds or changes and the test that pins it.

**Behaviour → pinning test**

1. `weftConflictStop` builds a `WeftStopError` with Branch, Paths, WayForward and Err.
   - Only `WayForward` is pinned: `failedpush_test.go:491` (`WeftConflictStopMergesAtTheRecordsRootAndPushesInTheAnchoredDir`) compares it exactly, with the merge at the weft root and the push in `anchoredDir`.
   - Branch, Paths and Err are not asserted by any row.
   - `settleUnmovedWeft` passes `ahead` and `outcome.Conflicts` into the extracted function, and `pull_integration_test.go:397` asserts only `errors.Is(err, ErrReplayConflict)` for the real stop. No test checks that real wiring (conflict paths, `ahead` list) reaches the way forward.
   - The Err and Branch pass-through is a mechanical move of the removed literal, so I rate this LOW.
2. `WeftConflictStopForTest` in `export_test.go` is a helper. No row of its own is needed.
3. The anchored-hub row for the records-conflict stop is the same row as 1.
   - It reaches the "verb directory call in an anchored hub" part of the answer key.
   - It is not a duplicate of `WeftDirtyStopCommitsInTheAnchoredDir`: that row pins the commit wording, this one pins the merge and push wording.
   - The row is also not covered by the `recordsConflictWayForward` unanchored rows at `failedpush_test.go:341-371`, which use non-anchored paths.
   - The want string is hand-built and duplicates the production wording, but the neighbouring rows do the same. It is not a finding.
   - `anchoredTask`, `anchoredWeft` and `anchoredDir` are defined at `failedpush_test.go:253-255`, and the row sits inside the same function (`TestFailedPushRecord`). They are in scope.
4. `newFailureFixtureAt` and `newPairFixtureAt` are parametrised by anchor.
   - `newFailureFixture` and `newPairFixture` delegate with `"."`, which is the previous behaviour, so other callers are unchanged.
   - The pass-through itself needs no test.
5. `AWeftPushThatMustReplayStopsOnAnAnchorChange`, row `OntoAnUpstreamCarryingAnotherMarkerSetsTheAnchorInTheAnchoredDir`.
   - The hub is anchored at `backend`, and the row asserts the full phrase `lym fabric anchor <worktree> --set sub` in `filepath.Join(p.warpPath, "backend")`, which pins the anchored directory.
   - The other two rows keep `"."` and still assert `lym fabric anchor` and the marker name as before.
   - The other two rows assert only a verb name or a marker name, not a directory. That predates the diff, so it is not a finding.
6. Scope check against the answer key, from the tests only.
   - The key asks for an anchored-hub row for the weft-dirty stop and the upstream-anchor stop of the steps not yet converted.
   - `WeftDirtyStopCommitsInTheAnchoredDir` (`failedpush_test.go:483`) predates the diff and covers the weft-dirty stop.
   - The upstream-anchor stop is pinned in `stopRows` (`AnchorStoppedPairWeftSetsTheAnchorInTheAnchoredDir`) and by the changed integration row. Both predate the diff or are the diff's own row.
   - I found no anchored row for the `weftAnchorStopBeforePull` (`anchorstop.go:171`) caller of `lymVerbDir`. I did not read that file, so I raise it as a candidate for the handler rather than a confirmed gap. It is a possible incomplete "each call" coverage, MEDIUM, class `scope`.
   - `originreconcile.go:392` (`stopWayForward`'s pull case) is pinned by `RewrittenCodeBranchPullsInTheAnchoredDir`.
   - `failedpush.go:299` (the `RepoPath` branch of `pushStep`) is pinned by `AnchoredCodeTransportPushesInTheAnchoredDir`.

**Findings**

- LOW, `design`, `failedpush_test.go:491`: the new row asserts only `WayForward`. The `Branch`, `Paths` and `Err` fields of the extracted stop are unpinned, and the real `settleUnmovedWeft` wiring is not asserted for the anchored way forward. Evidence: `pull.go:464-470`, `pull_integration_test.go:397`.
- MEDIUM (candidate), `scope`, `anchorstop.go:171`: I did not confirm that `lymVerbDir(codeWorktree)` there is pinned by an anchored-hub row. The handler should verify against the board entry's "each call". Evidence: grep output listing the call sites.

**Tool calls**
1. `grep` on `failedpush_test.go` for the anchored fixtures and in `*_test.go` for `ErrReplayConflict` / `reset --keep`: my context lacked where `anchoredTask`, `anchoredWeft` and `anchoredDir` are defined and which tests already assert the conflict stop.
