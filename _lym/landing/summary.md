# gitexec: stall test waits on remote readiness, not wall-clock

`TestRun_RemoteBounds` no longer fails when a loaded machine spends the 200 ms stall bound starting git.
Before, the silent-remote row killed git at the bound, and at a load average near 30 that came before the ext script wrote its count file.
The same row failed in the `heddle-render-ignores-anchor` gate run and in a `-count=1` rerun on an unrelated change.

## What changes

`gitexec` starts its stall check through a new package variable, `startStallTimer`, in `internal/gitexec/gitexec.go`.
Production behaviour is unchanged: it is `time.AfterFunc(bound, check).Stop`.
The variable exists only so a test can run the check on its own signal.

`internal/gitexec/export_test.go` gains `SetStallCheckForTest`, which runs each armed check once a `ready` callback reports true, instead of once the bound passes.
It also returns the bounds the checks were armed with, one per attempt.
`StartDefaultStallTimerForTest` reaches the production timer while the seam is replaced.

In `gitexec_unix_test.go`:

- The silent-remote row runs its check once the ext script has counted a new run, and uses a long remote deadline so git start-up has room.
  It also asserts a check was armed with the stall bound for each of the two attempts.
- The answering-remote row runs its check once git has traced the remote's first packet, so it proves the check does not kill a remote that answered.
  It asserts the check ran and was armed once.
- The row where the deadline is at or under the stall bound installs an always-ready check, so an armed check would fire and fail the row's assertions.
- A new row covers the production timer: it runs its check once the bound passes, and `stop` cancels a pending one.

## Design decisions

- The bound is not raised.
  The 200 ms was a stand-in for "the remote has been silent", so the test now waits for the readiness that bound approximated, and the 200 ms stays only as the value the check is armed with.
- The seam replaces the timer, not the check body, so the production check that stats the trace file and cancels the attempt still runs in every row.
- `SetStallCheckForTest` changes process-global state, so the rows stay serial, as the test's doc comment states.

## To check by hand

- `stall-timer-default-untested` (design, medium): the production `startStallTimer` is covered only by the new row in `gitexec_unix_test.go`.
  Confirm that row proves what the test seam bypasses, that the check runs after the bound and that `stop` cancels it.
- Not run here: `lym gate test ./internal/gitexec` under heavy load, the condition that caused the flake.
  Run it with the machine loaded to confirm the row now passes.
