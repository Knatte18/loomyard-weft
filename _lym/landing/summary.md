# Make timing tests load-proof

Tests that compare a measured time to a limit failed on a busy machine rather than on a code fault.
One of them, `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver`, cost a whole second gate pass under load.
This change rewrites the timing assertions found across the test suite so a loaded machine cannot fail them.
No production code changes.

## What changes

A test that proves a call does not wait on something now makes that thing outlast a generous one-minute deadline.
It does so with a longer sleep, a longer wait bound or a longer child lifetime.
A call that wrongly waited then fails however loaded the machine, and a correct call passes at any load.
Where an outcome already tells the two paths apart, the test asserts it instead of a time, and the time check goes.

- Longer stand-ins with a one-minute bound:
  `battencli` (`runRowCommand`), `cardverify`, `desktophook`, `dobbyengine`, `gatecli` detached-run, `gitexec`,
  `testcover`, `treadleengine`, `verifyrun`.
- Time check dropped for an outcome:
  - `gatecli` scenario: the watchdog's own quit line proves what ended the run.
  - `gatecli` late-launch call: the stand-in's exit 0 proves the call answered before its cap.
  - `githubclient` token chain: no timing assertion remains.
  - `githubclient` `TestAuthRT_HeldCallGetsItsFullDeadlineAfterRelease`: `context.Canceled` rather than `DeadlineExceeded`.
  - `reedengine` `EnsureSession`: the unchanged server pid proves the pre-started server was reused.
- `githubclient` `TestRunGHAuthTokenSeam_HonoursGhAuthTokenTimeout` now records the time left on the context at entry.
  That time only shrinks under load, so it tells the shortened timeout from the production one.

`pattern/PATTERN-test-speed.md` gains the rule and its one exception.

## Design decisions

- The pattern allows one exception: a cost budget no outcome can stand in for may compare a measured time to a bound.
  The bound must sit far above what load adds to the work, so only a gross regression reaches it.
- `startCostBound` in `internal/battencli/lifecycle_integration_test.go` is that exception, marked `testtiming:keep`.
  It rose from 100 ms to 30 s.
  It now catches a Command row that waits tens of seconds on a remote or a fixed delay, not a few milliseconds of overhead.

## To check by hand

- `startCostBound` still compares a measured time to a limit.
  The pattern's exception permits it, but a reviewer should confirm that a 30 s bound is a worthwhile guard
  and that the exception, not a release-controlled wait, is right for a start-cost budget.
- The `treadleengine` lingering-child tests (`gate_lingering_test.go`) use Windows `cmd` and `ping`.
  The change was not run on Windows.
