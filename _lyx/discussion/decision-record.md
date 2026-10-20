# Decision record — lyxbin timeout test independent of helper start time

## Goal

`TestRun_KillsTheGroupOnTimeout` in `internal/testkit/lyxbin` passes however long an endpoint scanner such as Cortex XDR holds the freshly built helper binary before it runs.

## Scope

- `internal/testkit/lyxbin/lyxbin.go`: split `Run` into the public timeout wrapper and an unexported context-driven body the test can end on its own signal.
- `internal/testkit/lyxbin/lyxbin_integration_test.go`: the timeout test ends the run only once the sleeper has recorded its pids, and the sleeper writes its pid file atomically.
- `Run`'s signature and observable behavior for its callers stay as they are.

## Decisions

### The run body ends on a context cause, not on `DeadlineExceeded`

Decision: an unexported `runUntilDone(ctx, bin, dir, args...)` holds `Run`'s body and treats a context whose cause wraps `ErrTimeout` as the timeout, answering exit code -1 and that cause wrapped with the arguments, directory and output.
`Run` builds its context with `context.WithTimeoutCause`, the cause being `ErrTimeout` wrapped with the timeout, so its error still names the timeout.
Rationale: the test then starts the "timeout" when the sleeper is up, by cancelling with `ErrTimeout`, so process start is outside the deadline, while `Run` stays a thin wrapper whose own clock the other callers keep.

### The test cancels once the pid file exists

Decision: the test runs `runUntilDone` with a `context.WithCancelCause` context and a goroutine that polls for the pid file on a short tick and cancels with `ErrTimeout` once it exists, or once a two-minute cap passes.
Rationale: no signal crosses the process boundary, so a short-tick poll with a deadline is the fitting wait;
the cap bounds a sleeper that never starts, which then fails on the missing pid file as before.

### The sleeper writes its pid file by rename

Decision: the sleeper writes its pids to a sibling temporary file and renames it onto the pid file path.
Rationale: the test reads the file as soon as it exists, so the content must appear whole.

## Constraints

- `PATTERN-test-speed`: no fixed real-time sleep stands in for the wait; the poll has a short tick and a deadline.
- `PATTERN-testkit`: the kit stays imported only from tests; no new export.
- `PATTERN-comment-line-breaks`: one sentence per comment line.

## Auto-mode assumptions

- The board entry names a 5 s timeout; the test on this branch uses 3 s, and the fix removes the dependence on either.
- The `sleep` child is a system binary the scanner already knows, so its start is not the delay.

## Open risks

- A scanner hold longer than the two-minute cap still fails the test, on the missing pid file.

## Acceptance criteria

- The group-kill test asserts `ErrTimeout`, exit code -1 and that neither recorded pid survives, with the run ended only after the pid file exists.
- `Run` still answers `ErrTimeout`, exit code -1, and an error naming the timeout when its command outlives it.
- The `integration` tier of `internal/testkit/lyxbin` passes.
