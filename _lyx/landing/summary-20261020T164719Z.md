# lyxbin: the group-kill test starts its timeout once the sleeper is up

`TestRun_KillsTheGroupOnTimeout` in `internal/testkit/lyxbin` failed on a host running Cortex XDR, which holds an executable with an unseen hash for 8-10 s before it runs.
The test's fixed 3 s timeout included the helper's start, so the run timed out before the sleeper wrote its pids (issue #767).

## What changes

- `Run`'s body moves into an unexported `runUntilDone(ctx, bin, dir, args...)`.
  It treats a context whose cause wraps `ErrTimeout` as the timeout: it kills the group and answers exit code -1 with that cause wrapped with the arguments, directory and output.
- `Run` keeps its signature and behavior.
  It builds its context with `context.WithTimeoutCause`, the cause being `ErrTimeout` wrapped with the timeout, so its error still names the timeout.
- `TestRun_KillsTheGroupOnTimeout` drives `runUntilDone` with a `context.WithCancelCause` context.
  A goroutine polls for the pid file on a 20 ms tick and cancels with `ErrTimeout` once it exists, so process start never counts against the timeout.
  A two-minute cap bounds a sleeper that never starts, which then fails on the missing pid file as before.
- The sleeper writes its pid file by rename, so the test never reads a half-written file.
- The new `TestRun_TimesOutNamingTheTimeout` pins `Run`'s own clock: a 200 ms run of the sleeper answers `ErrTimeout`, exit code -1 and an error naming `200ms`.
  It passes whether or not the sleeper starts in time, since either way it outlives the timeout.

## Design decisions

- The seam is unexported and its context is the test's to end, since no caller outside the package needs a context-taking `Run`.
- The timeout is recognised through `context.Cause` wrapping `ErrTimeout` instead of `DeadlineExceeded`, so a cancel-cause context and a timeout context take the same path.
- A longer fixed timeout, pre-warming the helper and retrying with growing timeouts were rejected: each still depends on how fast a fresh binary starts.

## Verification

`lyx gate test --tags integration ./internal/testkit/lyxbin` and the untagged `lyx gate test ./internal/testkit/lyxbin` pass, as does `lyx loom lint-comments`.
