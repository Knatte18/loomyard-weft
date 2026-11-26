# End every process a loomcli smoke fixture leaves under its temp dirs

A loom verb that reconciles the hub spawns a detached `lym board sync` in the fixture's board.
The sync runs on after the verb returns.
In `internal/loomcli`, `TestLoomFailedReedUpRefuses` never waited for it.
`waitForFixtureProcessesToExit` waited ten seconds at most and then returned silently, so a sync still running stayed alive.
When such a sync was still running at the package's end, `tmuxkit.Main` failed the tier1 run with `process … left running by the package`.
A rerun passed, because whether the sync outlives the package depends on timing.

## What changes

- `waitForFixtureProcessesToExit(t, patience)` waits up to `patience` for every process whose cwd is under the test's temp root.
  It then kills and logs each one still running.
  It fails the test only when a killed process still runs after a second `patience`.
  Callers pass `fixtureProcessPatience`, the ten seconds the old loop used.
- The temp root is resolved through `filepath.EvalSymlinks` before the scan.
  `/proc/<pid>/cwd` reports a resolved path, so an unresolved root that held a symlink would match no process.
- `newBadReedUpFixture` registers the cleanup itself, right after it copies the hub.
  This covers every subtest of `TestLoomFailedReedUpRefuses`, and `TestLoomResumeReconcilesTheHubConfigBeforeArming` drops its own call.
- `newWiredPairFixture` keeps its callers' explicit calls.
  Its tmux-tier users have their own teardown order, which this change leaves as it was.

## Test

`TestWaitForFixtureProcessesToExit_KillsAProcessThatOutlivesItsPatience` (tier1, Linux only) starts `sleep 60` under a subtest's temp dir with a 100 ms patience.
It asserts that the process was killed by SIGKILL once the subtest returns.
The defect and the fix both live in test-only fixture code, and the production code is unchanged.
No test can fail on the base's production code, as the record's "No test seam" decision states.

## Design decisions

- The cleanup kills a lingering process rather than failing the test on it.
  A slow board sync under load is not a defect of the test under it, and a kill keeps the package's end-of-run sweep clean.
  The kill is logged, so a process that always lingers still shows in the test's log.

## Merge danger

**Door:** two-way
**Blast radius:** loomcli
