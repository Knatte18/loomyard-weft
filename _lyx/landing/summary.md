# gitexec: bound a remote's first answer and retry a stalled call once

A git network call whose remote never answers used to hold a run until curl's connect timeout or `gitexec`'s ten-minute deadline.
Two batten runs hung this way on a TLS handshake GitHub never answered: git's low-speed limit covers a transfer, never the connection's setup.
A remote subcommand (push, fetch, pull, clone, ls-remote) now fails as a stall after 30 s without an answer and runs once more, so a dead connection costs seconds instead of minutes.

Every lyx git network call already goes through `internal/gitexec`, so no call site changes.

## What changes

- A remote subcommand runs with `GIT_TRACE_PACKET` pointed at a temp file under `lyxtemp`'s run directory, removed after the call.
  git writes the trace's first line only once the remote's first packet arrives, so a file still empty at the stall bound means the remote never answered.
  The call is then killed through the deadline's process-group kill.
- A killed stall is retried once, with a fresh deadline and stall bound, and logs one Warn naming the subcommand.
- A second stall returns a `*GitError` with the new `Stalled` field set and `Timeout` set to the stall bound.
  `IsTransportFailure` already reports a non-zero `Timeout` as a transport failure, so callers' transport handling takes it over unchanged.
- Local subcommands get no trace file and no stall bound.
- The package doc describes the stall bound, the retry and the trace override.

## Design decisions

- The retry is safe for a push because a stalled call received no ref advertisement and so changed nothing on the remote.
- Only a stall is retried; a deadline kill, a git exit and an exec failure are not.
- The stall check runs once, at the bound, and covers only the wait for the first answer.
  A connection that stalls after it is held by the low-speed limit and the deadline alone, and is not retried.
- A deadline at or under the stall bound, as `RunWithin` may pass, leaves the stall check unarmed, so such a call is killed once at its deadline as before.
- The pause hold stays outside the retry loop: the network gate is taken once per call, and both attempts' bounds start only when the call runs.
- An operator's own `GIT_TRACE_PACKET` is overridden for remote subcommands lyx runs.
- The 30 s bound is a judgment: long enough for GitHub's first answer on a slow link, short enough to save minutes.
- Out of scope: go-git network calls, the deadline and the low-speed limit.

## To check by hand

- Not run against a real network.
  The tests shorten the stall bound against local fake remotes; a dead connection to GitHub was not reproduced.
- A git build that writes no packet trace for some transport would see every call on it stall twice and fail.
  HTTPS and local remotes were checked to trace; SSH was not run here.
