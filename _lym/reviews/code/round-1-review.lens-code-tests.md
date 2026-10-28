```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the hub-pull diff adds or changes, the test that pins it (lens code-tests), then report findings.

**Per-behaviour list**

- Side order board, prime, beams in worktree order, each reporting moved or up_to_date: pinned by `TestPullHub_MovesEverySideOriginMovedThenReportsUpToDate` (`internal/fabricengine/hubpull_integration_test.go:83`). Its second pass pins the level hub, `up_to_date` for every side with no mutations.
- Task pairs are left out of the pull: the same test (line 87 adds a task pair, line 106 expects no side for it).
- Each pair's pull runs inside its own lock: the same test via `recordingPairLock` (line 35), and the CLI wait test (`internal/fabriccli/hubpull_integration_test.go:115`).
- Lock held for one pair only, never across sides: none. The recording seam checks only that each pull sits inside its own lock. Nothing fails if a release is delayed to the end of `PullHub`, and `locked` order doesn't detect it. A CLI test shows no lock held after the whole pull, not between sides.
- A stop leaves the other sides pulled: `TestPullHub_StoppedSideLeavesTheOthersPulled` (line 141), a dirty prime with the board and a beam still moving.
- Dirty prime reports `ErrWarpDirty` text as detail: line 169 of the same test.
- No upstream means `up_to_date` with no lock taken: lines 172 and 175 of the same test.
- Bolt pull under the board write lock, lock released after: `TestBoltPull_HoldsTheBoardWriteLockWhileItPulls` (line 186). It probes the lock without a clock.
- Bolt skip (dirty board) reported `stopped` with detail, and unreachable records origin reported `stopped`: `TestPullHub_StopsABoardSideAndAPairSideWithTheirDetail` (line 308).
- `SkipGit` leaves every side up to date: `TestPullHub_HonorsSkipGitWithEverySideUpToDate` (line 253). It covers `Bolt.Pull`'s early return too.
- Unreadable origin record stops only that beam: `TestPullHub_UnreadableBeamRecordStopsOnlyThatBeam` (line 275).
- `HubPullResult.Stopped`: pinned at lines 110 and 166.
- CLI `--hub` flag, error envelope with every side and exit 1, the error naming the stopped side: `TestRunCLI_PullHubFromATaskPair` (CLI test file, line 27). It also runs from a task pair.
- `--hub` waits on a held parent lock, owner run id `fabric-pull`, and a cancelled wait stops only the prime side: `TestRunHubPull_WaitsForAHeldParentLock` (CLI test file, line 115).
- `acquireBranchLock` now takes a verb; `anchor` keeps the run id `fabric-anchor` and its wait text: no new test. The diff shows no test change for anchor, so this relies on existing anchor tests, which I did not open.
- Plain `fabric pull` without `--hub` unchanged: no new test in the diff, which fits the scope, and existing pull tests cover it.
- `hubPullSidesMap` and `hubPullStopError`: no direct tests, which is correct. The CLI tests cover them at the verb surface.

**Findings**

- LOW, design: `internal/fabricengine/hubpull_integration_test.go:35-55`. No test pins that a pair's lock is released before the next side is pulled. The decision record says the lock is never held across sides. Fix: in `recordingPairLock`, track a held count and fail with `t.Errorf` if a lock is taken while another is held.
- LOW, design: `internal/fabricengine/bolt.go:243-256`, which no test pulls through. `Bolt.Pull` on a Bolt with no upstream and the board write lock failing are uncovered. The no-upstream case is a documented contract ("does nothing"). Fix: add a row to the board tests with a Bolt with no upstream, expecting an `up_to_date` board side. The lock failure is a boundary error, so skip it.

No test-economy finding. I found no duplicate test, no per-helper test and no stencil-prose assertion. The two CLI test files cover distinct behaviours: the task-pair run and the lock wait.

**Tool calls**
1. `grep` of test function names plus `git diff` of `export_test.go`: names and the test seam, which my context lacked.
2. Read of engine test lines 83–342: test bodies.
3. Read of CLI test lines 27–181: test bodies.
4. Read of engine test lines 30–58: the `recordingPairLock` helper body, to see whether it checks release order.
