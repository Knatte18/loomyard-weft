# Decision record — git network calls get a stall bound and one retry

## Goal

A git network call whose remote never answers fails within seconds and is retried once, instead of holding a batten run for minutes on a dead connection.

## Scope

- `internal/gitexec`: the remote-subcommand path of the exec core, `GitError`, the package doc and its tests.
- Every git network call lyx makes already goes through `gitexec` (push, fetch, pull, clone and ls-remote are its remote subcommands), so no call site changes.
- Out of scope: go-git network calls, the ten-minute deadline and the low-speed limit, which stay as they are.

## Decisions

### A stall is no answer from the remote within the stall bound

A remote subcommand runs with `GIT_TRACE_PACKET` pointed at a temp file under `lyxtemp`'s run directory.
When the stall bound, 30 s, passes and the file is still empty, the remote has sent no packet, and the command is killed through the same process-group kill the deadline uses.
Rationale: git's low-speed limit covers a transfer, never curl's connect phase, so a TLS handshake that gets no answer, as seen, runs to curl's connect timeout or lyx's ten-minute deadline.
The packet trace's first line is written only once the remote's first answer arrives, over HTTPS, SSH and a local remote alike, so an empty trace is a sound sign of a stall.
Bound: the check runs once, at the stall bound, and covers only the wait for the remote's first answer;
a connection that stalls after it is still bounded only by the low-speed limit and the deadline.
A deadline at or under the stall bound leaves the stall check unarmed.

### A stalled call is retried once

A remote subcommand killed for a stall runs once more, with a fresh deadline and stall bound, and a second stall returns a `*GitError` with `Stalled` set and `Timeout` set to the stall bound.
Rationale: a stalled call never received the remote's ref advertisement, so it updated nothing on the remote, and running it again is safe for push as for a read.
Bound: one retry, only on a stall; a deadline kill, a git exit and an exec failure are never retried.
The retry logs one Warn naming the subcommand.

### The stall is a transport failure

`IsTransportFailure` already reports a `*GitError` with a non-zero `Timeout` as a transport failure, and a stalled `*GitError` carries one, so callers' existing transport handling takes the second stall over.

## Constraints

- `PATTERN-lyx-temp-root`: the trace file lives under `lyxtemp`'s run directory and is removed after the call.
- `PATTERN-gitexec-checked-call`: no new raw git spawn.
- `PATTERN-test-speed`: the stall bound is shortened by the test, never waited out.
- `PATTERN-log-levels`: one Warn per retry, at most one retry per call.

## Auto-mode assumptions

- 30 s is long enough for GitHub's first answer on a slow link and short enough to cost seconds instead of minutes.
- An operator's own `GIT_TRACE_PACKET` is overridden for remote subcommands lyx runs; the package doc states it.

## Open risks

- A git build that writes no packet trace for some transport would see every call on it stall twice and fail; HTTPS and local remotes were checked to trace.

## Acceptance criteria

- A remote that accepts the connection and never answers fails as a stall after two attempts, each killed at the stall bound, and `IsTransportFailure` reports it.
- A remote that answers and then hangs is not killed at the stall bound, is not retried, and is killed at the deadline.
- A deadline at or under the stall bound kills a hung remote once, at the deadline, as before.
- Local subcommands get no trace file and no stall bound.
- The package doc describes the stall bound, the retry and the trace override.
