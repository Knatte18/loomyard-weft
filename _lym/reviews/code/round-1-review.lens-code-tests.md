```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds, I name the test that pins it, or none, and apply the rubric's "Test economy".

**Per behaviour: pinning test**
1. Side order Bolt, prime, beams: `TestPullHub_MovesEverySideOriginMovedThenReportsUpToDate` pins it with one beam. Order among several beams is not pinned, since no test adds a second beam.
2. Bolt moved: the same test. The CLI test's `EveryMovedSidePulled` row also asserts `board moved`.
3. Prime moved: the same engine test (file present, locked pair recorded). The CLI row repeats it at the envelope level.
4. Beam moved: the same engine test, via `moveBeamOnOrigin`.
5. Level hub, every side `up_to_date`, no mutations: the second `PullHub` call in the same test. There is no CLI-level exit-0 check on a level hub, which is acceptable because the first CLI row pins exit 0.
6. A stopped side leaves the others pulled:
   - `TestPullHub_StoppedSideLeavesTheOthersPulled` covers the dirty prime, with Bolt moved before it.
   - The side after the stop is the no-upstream beam, which takes the shortcut and pulls nothing. So no test shows a side after a stop actually pulling and moving. Bolt stopping and then prime and beams still running is also unpinned.
7. A stop gives a non-zero exit, an error envelope with every side's entry, and an error naming the stopped side: the CLI row `DirtyPrimeStopsAndFailsTheVerb`.
8. The stopped prime's detail is the `ErrWarpDirty` text: the engine stopped-side test.
9. A beam with no upstream is `up_to_date` and takes no lock: the engine stopped-side test (detail text and the locked list `[prime]`).
10. Each pair's pull runs inside its own lock: `recordingPairLock`, which checks the file is absent at lock and present at release, in the engine moved test.
11. Locks are released after the pull (never held across sides): the CLI test checks no parent locks remain. The engine `defer release()` is not pinned separately, which is acceptable.
12. Bolt's pull runs under the board write lock: `TestBoltPull_WaitsForTheBoardWriteLock`.
13. The pair pull waits while another owner holds the branch's parent lock, then runs once it is released (acceptance criterion 5): **none**. The CLI test only checks that the lock is taken and released uncontended. `acquireBranchLock`'s waiter, its holder-naming stderr notice with the `verb` parameter, the `fabric-pull` run id, and the Ctrl-C end of the wait are all untested.
14. Task pairs excluded from the beam list: only implicit. The CLI test's acting task pair is a non-beam pair that does not appear in `sides`, but no engine test puts a non-beam pair beside a beam.
15. A beam is honored whatever machine owns it: none. `Main` and `Prunable` skipping and the `ReadOriginFor` error path are also unpinned.
16. `PullHub` returning an error (unreadable worktree list or origin record) with the sides already pulled kept: none.
17. `!pullRes.WeftPulled` gives `stopped` with the "origin could not be reached" detail: none.
18. `pullBoardSide`'s stop branches (`skip != ""` and `err != nil`): none. The way-forward detail reaching the report from a Bolt skip is unpinned.
19. `opts.SkipGit` making `Bolt.Pull` do nothing: none.
20. `--hub` leaves plain `lym fabric pull` unchanged: no new test, and none is needed beyond the existing pull tests. The role guard refusing `--hub` is likewise unchecked.
21. `acquireBranchLock` parameterized by `verb`: the CLI hub-pull test covers `"pull"`. I did not check whether an existing anchor test asserts the old run id or wait text. `anchorLockOwnerRunID` was removed, so a compile failure would show it.

**Findings**

- **MEDIUM, design (behaviour added with no coverage).** The parent-lock wait has no test (item 13). Evidence: `internal/fabriccli/hubpull_integration_test.go:30-72` and `internal/fabricengine/hubpull_integration_test.go:20-45`, where `recordingPairLock` is a fake that never contends. `acquireBranchLock` in `internal/fabriccli/anchordeps.go` (the `parentlock.Waiter` and `RunID: "fabric-" + verb`) is exercised only uncontended. Acceptance criterion 5 is therefore unpinned.
- **MEDIUM, design (behaviour added with no coverage).** No test shows a side after a stopped side still pulling and moving. In `internal/fabricengine/hubpull_integration_test.go:~148-175` the only side after the stop is the no-upstream beam, which returns before any pull. A regression that ended the loop at the first stop would still pass, except a regression that skipped the beam entirely. The beam-after-stop row should use a pushed beam that origin moved. A Bolt stop followed by an unaffected prime is also unpinned.
- **LOW, design (uncovered added behaviour).** The `WeftPulled == false` stop in `pullPairSide` (`internal/fabricengine/hubpull.go:137`), `pullBoardSide`'s skip and error stop branches (`hubpull.go:~86-92`), and `PullHub`'s error return from `localBeamLocations` (`hubpull.go:~72-74`) have no test row. The skip branch is where the way-forward detail comes from for Bolt.
- **LOW, design (uncovered added behaviour).** Beam selection (`localBeamLocations`, `hubpull.go:~152-178`) is untested for a non-beam task pair beside a beam, for a beam owned by another machine (the "whatever machine owns it" decision), and for several beams in `git worktree list` order. One row in the moved test with a second pair would cover the first and third.
- **LOW, design (test determinism).** `TestBoltPull_WaitsForTheBoardWriteLock` (`internal/fabricengine/hubpull_integration_test.go:~195`) waits a fixed real 200ms with `time.After`, which the `testing` skill's "no fixed real-time sleep" rule forbids. The comment explains that a blocked flock gives no signal. The test cannot fail falsely but is slow, and it passes vacuously if `Bolt.Pull` ignored the lock and merely ran slowly.
- **NIT, design (test economy).** The CLI row `EveryMovedSidePulled` repeats the engine moved test's board and prime coverage. It adds exit 0 and the lock-released check, and the dirty row already checks the envelope shape and the lock release. This is not worth a finding by the "name the covering test" standard, which the engine test would meet. I raise it only as a possible row merge.

Nothing else duplicates an existing test: both new files are new, and I opened no pre-existing `Bolt` or `acquireBranchLock` tests.

**Tool calls:** none. The diff and its context were already in the manifest.
