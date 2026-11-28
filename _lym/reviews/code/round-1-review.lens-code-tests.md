```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test row that would fail if it broke.

**Behaviours and the tests that pin them**
1. `worktreeGone` reads a standing worktree (one with `.git`) as not gone: `TestWorktreeGone_ReadsWhetherTheWorktreeStands`, top-level assertion. Pinned.
2. `worktreeGone` reads a missing path as gone: same test, row `TheDirectoryIsGone`. Pinned.
3. `worktreeGone` reads a leftover directory without `.git` as gone: same test, row `TheDirectoryStandsWithoutItsGitEntry`. Pinned.
4. `worktreeGone` returns an error when the stat fails other than not-exist, for example ENOTDIR when the path is a regular file: **none**.
5. `starter.start` skips a gone pair, reading and spawning nothing: `TestStart/AGonePairReadsNothingElse`, through `wantNoReads` and `wantSpawn` false. Pinned.
6. `starter.start` with `ifAhead` writes no ledger entry for a gone pair: `TestStart/IfAheadWithAGonePairWritesNothing`. Pinned.
7. The gone read does not apply to non-pair targets: `TestStart/AGoneReadDoesNotApplyToThePrime`. Board and landing share the same `t.Pair` branch. Pinned.
8. A failing gone read is returned and starts nothing: `TestStart/GoneReadFails`. Pinned.
9. `newStarter` wires `pairGone` to the real read: `TestTargetOf_AWorktreeNoLongerStandingIsAPairThatStartsNoPusher`, through the real `StartIfAhead`. The test env sets no `FABRIC_SKIP_*` (`gitkit.HermeticGitEnv` sets only git config), so without the wiring `targetAhead` would run git on the gone path and fail. Pinned.
10. `TargetOf` returns the pair target with nil error for both kinds of removal: same `TestTargetOf_...` test. On the old code `IsPrimeWorktree` fails and the test's `Fatalf` fires. Pinned.
11. `TargetOf` returns a pair target together with the stat error when the stat fails: **none** (same gap as 4).
12. `TargetRefused` reads a gone pair as not refused: same `TestTargetOf_...` test. Pinned.
13. `TargetRefused` propagates a gone-read error: **none** (same gap as 4).
14. `PruneLedger` drops the entry of a leftover directory without `.git`: `TestPruneLedger`, entry `leftover`. Pinned.
15. `PruneLedger` keeps an entry whose standing worktree holds `.git`: `TestPruneLedger`, entry `standing`, which now writes `.git`. Pinned.
16. `PruneLedger` keeps the entry on a failed worktree read and returns the error, in a branch the diff rewrote: **none** (no row in `TestPruneLedger` makes the read fail).
17. `NewDeps.WorktreeGone` now uses the new definition: the function is pinned by items 1–3; the one-line wiring has no direct row. No finding: it calls a read that is already pinned.

**Findings**
- LOW, design, `internal/runpusher/targetreads.go:86-95`: the error path the diff adds to `worktreeGone` has no test row. That covers a stat failure other than not-exist, such as ENOTDIR when the path is a regular file, which `os.Lstat` returns as an error that is not `fs.ErrNotExist`. The same gap leaves the propagation through `TargetOf` (`internal/runpusher/target.go:77-79`) and `TargetRefused` (`internal/runpusher/targetreads.go:101-105`) unpinned. It also leaves `PruneLedger`'s rewritten failure branch unpinned: "a failed worktree read keeps its entry and is returned" (`internal/runpusher/targetreads.go:140,153-154`). Evidence: the `removals` table in `internal/runpusher/seams_test.go:65-71` has only the missing-path and missing-`.git` rows. `TestPruneLedger`'s entries (`internal/runpusher/start_test.go:246-252` in the diff) hold no worktree whose read fails. Fix: add a row to `TestWorktreeGone_ReadsWhetherTheWorktreeStands` where the path is a regular file and an error is expected. Add an entry to `TestPruneLedger` whose `Worktree` is a regular file, expecting the entry kept and an error returned.

Tool calls:
1. Listed the runpusher test functions, the env setup and `pairTargetIn`: the context lacked the test files' existing rows and fixture.
2. Read `testmain_test.go`, the `TestStart` harness and the hermetic env: needed to know whether `FABRIC_SKIP_*` would make the real `StartIfAhead` assertion pass trivially.
3. Grepped `gitkit.HermeticGitEnv` for `SKIP`/`Setenv`: confirmed it turns no pushes off.
