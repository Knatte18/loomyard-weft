# Bound TestWatchdogReap's success waits by a fixed 30 s

`TestWatchdogReap` in `internal/reedcli` failed on a host running Cortex XDR.
The `NeverEnteredOrphanIsStillReaped` step waited `DiscoveryCycle*OrphanGoneCycles*20`, 1.8 s, for the reap loop to remove an orphaned tmux session.
Cortex XDR holds each new process image, and every discovery cycle spawns tmux and git, so a cycle runs far longer than the compressed 30 ms cadence and the window closed before the reap landed.

Every wait for an outcome a step expects to succeed now uses one named constant, `reapSuccessWait` (30 s), in `internal/reedcli/watchdogreap_integration_test.go`.
That covers the orphan-session-gone waits in each step, and the wait for a fresh session to be listed in the in-flight step, which was `DiscoveryCycle*10`.
Only the test file changes.

Closes #769

## Design decisions

- `waitForCondition` polls, so a fast host returns as soon as the condition holds and never pays the bound.
  The constant lengthens only how long a broken reap takes to fail; it lets no wrong outcome pass.
- The bound is a deadline polled against and never slept, so `PATTERN-test-speed` still holds.
- `compressedReapTiming` keeps its compressed cadence and the production `OrphanGoneCycles`, so the test still proves the consecutive-cycle rule.
- Two checks keep their short bounds because they assert time running out, not success.
  The "still live before `OrphanGoneCycles` cycles" check in `EndToEndAndSiblingSurvives` passes on a stall rather than failing.
  The cancel deadline in `LoopStaysLiveDuringReap` bounds an in-process select with no spawn on its path.
- A host slower than 30 s still fails; raising the constant is a one-line change.

## To check by hand

- `lyx gate test --tags tmux ./internal/reedcli` was not run for this description; run it to confirm `TestWatchdogReap` passes.
- The failure was seen only under Cortex XDR, so the fix is unconfirmed on such a host until a run there passes.
