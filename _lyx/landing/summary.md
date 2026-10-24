# Git network calls fail on a stall within 30 s and retry once

A git network call to a remote that never answers used to hold a run for minutes.
git's `http.lowSpeedLimit`, which `gitexec` already sets, covers a transfer but not the connection's setup.
A TLS handshake that gets no answer therefore ran to curl's connect timeout or to lyx's ten-minute deadline.
Two batten runs hung this way, one on an ls-remote and one on a push.

## What changes

Every remote subcommand `gitexec` runs (push, fetch, pull, clone and ls-remote) now runs under a stall bound of 30 s, on top of the existing deadline and low-speed limit.
No call site changes, since every git network call lyx makes already goes through `gitexec`.

- The remote subcommand runs with `GIT_TRACE_PACKET` pointed at a temp file in `lyxtemp`'s run directory, and the file is removed after the call.
  git writes the file's first line only once the remote's first packet arrives.
  This was checked against real HTTPS ls-remote and push to GitHub.
- When the stall bound passes and the file is still empty, the call is killed through the same process-group kill the deadline uses, and run once more with fresh bounds.
- A second stall returns a `*GitError` with the new `Stalled` field set and `Timeout` set to the stall bound.
  It renders as `git <args>: no answer from the remote within 30s, twice`, and `IsTransportFailure` reports it as a transport failure with no change, so callers' existing transport handling takes over.

## Design decisions

- The retry is safe for a push: a stalled call never received the remote's ref advertisement, so it updated nothing on the remote.
  For the same reason, only a stall is retried. A deadline kill, a git exit and an exec failure are not.
- The stall check runs once, at the bound, and covers only the wait for the remote's first answer.
  A connection that stalls after the first answer is still bounded only by the low-speed limit and the deadline.
- A deadline at or under the stall bound, as `RunWithin` may pass, leaves the stall check unarmed, so the shorter deadline governs as before.
- The trace goes to a file path rather than an inherited file descriptor, because Go's `ExtraFiles` is unsupported on Windows.
- An operator's own `GIT_TRACE_PACKET` is overridden for remote subcommands, and the package doc says so.
- The retry logs one Warn naming the subcommand.

## Tests

`TestRun_RemoteBounds` gains two rows driven through git's `ext::` transport, with the stall bound shortened:
- A remote that never answers is killed at the stall bound and run exactly twice, and fails as a stalled transport failure.
- A remote that sends one packet and then hangs is not stalled. It is killed once, at the deadline, with no retry.

`TestGitError_Error` gains the stalled rendering.
