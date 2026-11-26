# End every process a loomcli smoke fixture leaves under its temp dirs

A loom verb that reconciles the hub spawns a detached `lym board sync` in the fixture's board,
and the sync runs on after the verb returns.
In `internal/loomcli`, `waitForFixtureProcessesToExit` waited at most ten seconds and then returned silently,
and `TestLoomFailedReedUpRefuses` never called it.
A sync still running when the package's tests ended made `tmuxkit.Main` fail the run
with `process … left running by the package`, intermittently and unrelated to the change under test.

Closes #264

## What changes

Only test files change.

- `waitForFixtureProcessesToExit(t, patience)` waits up to `patience` for every process whose cwd is under the test's temp root.
  It then kills and logs each one still running, and does the same for one that appears after a kill, a further `patience` later.
  It fails the test only when every process still running after that has already been killed.
  Callers pass `fixtureProcessPatience`, the ten seconds the old loop used.
- The temp root is resolved through `filepath.EvalSymlinks` before the scan.
  `/proc/<pid>/cwd` reports a resolved path, so a root reached through a symlink would match no process.
- `newBadReedUpFixture` registers the cleanup itself, which covers `TestLoomFailedReedUpRefuses`.
  `TestLoomResumeReconcilesTheHubConfigBeforeArming` drops its own call.
- `registerBootstrapTeardown` registers the cleanup before its own kills, so it runs after them.
  This covers the tier2 tests that use the teardown, and `TestSmokeWatchdog_StartSpawnsTheDaemon` drops its own call.
- The tier1 callers of `newWiredPairFixture` register no teardown and keep their explicit calls.

## Design decisions

- The cleanup kills a lingering process instead of failing the test on it.
  A slow board sync under load is not a defect of the test that spawned it.
  The kill is logged, so a process that always lingers still shows in the test's log.
- A process first seen after a kill no longer fails the test at once: it is killed and given its own `patience`.
- `TestWaitForFixtureProcessesToExit_KillsAProcessThatOutlivesItsPatience` (tier1, Linux only)
  starts `sleep 60` under a subtest's temp dir with a 100 ms patience, and asserts that SIGKILL ended it.
  The defect and the fix both live in `_test.go` files, so no test can fail on the base's production code.

## To check

- Tier2 callers of `newWiredPairFixture` leaving the detached board sync running,
  at `registerBootstrapTeardown` in `internal/loomcli/smoke_test.go`.
  The diff registers the cleanup there; confirm that it covers each of those callers.
- No test exercises the path where a new process appears after the first kill.

## Merge danger

**Door:** two-way
**Blast radius:** loomcli
