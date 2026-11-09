# Make tests that compare a measured time to a limit load-proof

Tests that proved "this call does not wait on X" did it by comparing the call's real elapsed time with a bound a few seconds above the normal case.
A busy machine pushes the call past such a bound, so the test fails when the code is fine.
This change removes every such comparison: either the thing the call must not wait on now outlasts a generous one-minute deadline, or the test asserts an outcome that already tells the two paths apart.

## What changes

Each thing the call must not wait on now outlasts a one-minute deadline:

- `internal/verifyrun` `TestRun_Cancellation`: the command sleeps 300 s, and the run must return within a minute of the cancel. The bound was 2 s.
- `internal/desktophook` `TestAfterWindow`: the hanging command sleeps 300 s, and each row must return within a minute. The bound was 10 s.
- `internal/gitexec` `TestRun_RemoteBounds`: the hung remote already sleeps 300 s, and the bound is now a minute instead of 10 s.
- `internal/testcover` `TestRunAlone`: the stand-in's pipe-holding child sleeps five minutes, and the test's cleanup kills it. The row allows a minute; it allowed 4 s against a 5 s child.
- `internal/dobbyengine`, the fresh-start refusal subtest: `ReexecLockWait` is ten minutes, and the bound is a minute instead of 2 s against a 3 s wait.
- `internal/cardverify`, the cancelled-line subtest: the sleeper sleeps 300 s, and the bound is a minute instead of 10 s.
- `internal/battencli` `stepRunRowCommand_ReportsATimeoutAndAnExitCode`: the command sleeps 300 s, and the bound is a minute instead of 3 s.
- `internal/gatecli` detached-front tests:
  - the failed-launch call gets a ten-minute claim bound and a one-minute limit, where the limit was 4 s;
  - the waiting call whose runner gets SIGTERM gets a ten-minute cap and a one-minute limit, where the cap was 60 s and the limit 30 s.
- `internal/treadleengine` `TestExecGateCommand_LingeringChildDoesNotHangPastWaitDelay`, which is Windows-only: the pings run about two minutes and the bound is a minute. The pings ran about 19 s against a 15 s bound.

These tests drop the time check and assert an outcome that already tells the two paths apart:

- `internal/reedengine` `TestEnsureSession_BootedTrueOnColdSessionFalseOnWarm`: a boot that waited out the stale-holder grace would replace the pre-started server, so the existing server-pid check proves the reuse.
- `internal/githubclient`:
  - `TestResolveToken_Chain` drops its 1 s check; the faked `gh` seam has nothing to wait on.
  - `TestRunGHAuthTokenSeam_HonoursGhAuthTokenTimeout` has the fake record the time its context has left at entry. It asserts that time is within the shortened timeout. The value only shrinks under load, and it still catches an override that does not apply.
  - The "cancelled after release" subtest relies on `context.Canceled`, since waiting out the deadline would give `context.DeadlineExceeded`.
- `internal/gatecli`:
  - The late-runner test relies on exit 0, because a call that reached its cap answers `SlotBusyExit`. Its claim bound grows from 4 s to a minute, so a runner that starts 2 s late is claimed in time even on a loaded machine.
  - The busy-spin test relies on the watchdog's own 1 s quit line rather than a one-minute elapsed bound that equalled the tier's timeout.

`pattern/PATTERN-test-speed.md` gains the rule under "Time is injectable": no test compares a measured real time to a bound a loaded machine can reach.

## Design decisions

- The idea's own example, `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver` in `internal/loomcli`, was already rewritten on main (in 2ad39d26) to hold the stub driver on a release file. Nothing here touches it.
- The tests use a one-minute deadline to match the 60 s `launchDeadline` that test uses. The cost: a regression that does wait now fails only once the outlasting duration ends, up to ten minutes for the dobby subtest, rather than in seconds.
- Most of these tests let X run past the deadline instead of holding it on a release file. A release file would make a regression hang until the gate's timeout, unless every test also gained a goroutine and a select. A sleep that outlasts the deadline fails on its own and needs no restructuring.
- `startCostBound` in `internal/battencli` rises from 100 ms to 2 s. A cost budget cannot be fully load-proof. The new bound still catches a Command row that waits seconds on a remote or a fixed delay, and it sits far above what scheduling adds on a loaded machine. A cost regression of tens of milliseconds is no longer caught; the batten-start research measurements cover that.
- Left unchanged:
  - Lower bounds (`elapsed < X`), because load can only lengthen the elapsed time.
  - Comparisons on a fake clock, such as those in `shuttleengine`, `creelengine`, `shedverbs`, `tenderengine` and `hibernateengine`.
  - The `seatengine` matches, which count send attempts rather than time.
  - The run-dir age precondition in `shuttleengine`'s `startup_test.go`, which has a 60 s margin over an operation of milliseconds.
- One race of the same kind remains: in `internal/gatecli`, the card-call test whose runner starts 2 s late against a 4 s claim bound. It compares no measured time, so it falls outside this entry; it can still fail on a heavily loaded machine.

## To check by hand

- The Windows-only `treadleengine` test was compiled for Windows (`GOOS=windows go vet`) but not run.
