# lyxbin: start the group-kill test's timeout once the sleeper is up

`TestRun_KillsTheGroupOnTimeout` in `internal/testkit/lyxbin` failed on machines where an endpoint scanner such as Cortex XDR holds a freshly built helper binary for several seconds before it runs.
The sleeper then never wrote its pids inside the test's `Run` timeout.
The test now ends the run only after the sleeper has recorded its pids, so process start no longer counts against any deadline.

Closes #767

## Changes

- `Run` keeps its signature and observable behavior.
  Its body moves into an unexported `runUntilDone(ctx, bin, dir, args...)`, and `Run` is a thin wrapper that builds the timeout context.
- `runUntilDone` treats a context whose cause wraps `ErrTimeout` as the timeout: it kills the group and answers exit code -1 with that cause, wrapped with the arguments, directory and output.
  `Run` builds its context with `context.WithTimeoutCause`, so its error still names the timeout.
- The group-kill test runs `runUntilDone` under a `context.WithCancelCause` context.
  A goroutine polls for the pid file on a 20 ms tick and cancels with `ErrTimeout` once it exists.
- The sleeper writes its pid file to a sibling temporary file and renames it into place, so the test never reads a partial file.
- A new `TestRun_TimesOutNamingTheTimeout` covers `Run` itself: `ErrTimeout`, exit code -1 and an error naming the timeout, whether or not the sleeper has started.

## Design notes

- The poll is capped at two minutes, so a sleeper that never starts fails the test on the missing pid file instead of hanging.
  A scanner hold longer than the cap still fails the test.
- No signal crosses the process boundary, so a short-tick poll with a deadline is the wait used, with no fixed real-time sleep standing in for it.
- The `sleep` child is a system binary the scanner already knows, so its start is not treated as a source of delay.
- The error text from `Run` changes in wording only: it now reads `... after <timeout>` through the cause, followed by the arguments, directory and output.

## Not run by hand

The fix was written without the scanner present.
Running the `integration` tier of `internal/testkit/lyxbin` on a machine with Cortex XDR active would confirm it.
