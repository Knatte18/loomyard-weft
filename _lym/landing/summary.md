# gitexec's stall test fires its check when the remote is ready

`TestRun_RemoteBounds`' row "a remote that never answers is killed at the stall bound and run once more" set a 200 ms stall bound under a 1 s deadline.
At a load average of about 30, starting git and its `ext::` script took longer than 200 ms, so the stall check killed git before the script wrote its count file, and the row failed in packages the change under test never touched.

## What the 200 ms is meant to prove

The bound's value carries no timing claim.
A stall bound under the deadline arms the stall check, and the check kills an attempt whose packet trace is still empty, so the row proves three things: a silent remote gets the stall error with the stall bound as its `Timeout`, the call runs once more, and the second stall ends the call.
None of those depends on how long git takes to start, so the test no longer lets start-up time race the check.

## The change

- `internal/gitexec/gitexec.go`: the stall timer is started through a package variable, `startStallTimer(bound, tracePath, check)`, which in production is `time.AfterFunc(bound, check).Stop`.
  It takes the attempt's trace path, which production ignores, so a test can wait on the trace.
  Production behavior is unchanged.
- `internal/gitexec/export_test.go`: `SetStallCheckForTest(t, ready)` replaces the timer with one that polls `ready(attempt, tracePath)` and runs the check once it reports true, or stops when the attempt ends.
  It returns the bounds the checks were armed with.
- `internal/gitexec/gitexec_unix_test.go`:
  - The silent row runs each attempt's check once the `ext::` script has counted that attempt's run, and asserts the check was armed twice with the stall bound.
    It also raises its own remote deadline to 10 s, so a slow start-up cannot reach the deadline first.
    The row still ends at the stall check, so it runs no longer.
  - The answering row had the same race the other way round: if the check fired before git traced the remote's packet, the row would see a stall.
    It now runs the check once the trace is non-empty, which proves the check spares a remote that answered, and asserts the check was armed once with the stall bound.
  - The unarmed row, where the stall bound equals the deadline, now asserts the check was never armed, rather than only inferring that from the error.

## Design decisions

- The fix moves the trigger rather than raising the bound, as the board entry asks: a higher bound only lowers the chance of failing under load.
- The remaining wall-clock bound is the 1 s package deadline in the hung, `RunWithin`, unarmed and answering rows.
  Those rows assert a kill at the deadline, so they still need git and its script to start within it.
  That leaves five times the headroom the 200 ms bound had, and lifting it would make each row wait out a longer deadline.
