# gitexec: kill a remote git call that gets no answer in 30 s, retry once

A git network call whose remote never answers now fails within seconds and runs once more, instead of holding a run for minutes on a dead connection.
This was seen on two batten runs: a `git ls-remote` and a `lyx fabric` push each sat on a TLS opening that GitHub never acknowledged, for over two minutes.
Git's low-speed limit covers a transfer, never the connection's setup, so nothing bounded the wait short of curl's connect timeout or the ten-minute deadline.

Every git network call lyx makes already goes through `internal/gitexec`, so no call site changes.

## What changes

- A remote subcommand (push, fetch, pull, clone, ls-remote) runs with `GIT_TRACE_PACKET` pointed at a file under `lyxtemp`'s run directory, removed after the call.
  git writes the trace's first line only once the remote's first packet arrives.
  When the 30 s stall bound passes and the file is still empty, the call is killed through the existing process-group kill.
- A call killed for a stall runs once more with a fresh deadline and stall bound, and logs one Warn naming the subcommand.
- A second stall returns a `*GitError` with the new `Stalled` field set and `Timeout` set to the stall bound.
  `IsTransportFailure` already treats a non-zero `Timeout` as a transport failure, so callers' existing handling takes it over.
  `GitError.Error` renders it as "no answer from the remote within 30s, twice".
- `doc.go` gains a "Stall bound" section.

## Design decisions

- **Retry is safe for a push.**
  A stalled call never received the remote's ref advertisement, so it changed nothing on the remote.
- **Only a stall retries.**
  A deadline kill, a git exit and an exec failure are never retried; there is at most one retry per call.
- **The stall check covers only the wait for the first answer.**
  A connection that stalls after it is still held by the low-speed limit and the deadline alone.
- **`RunWithin` with a deadline at or under the stall bound leaves the check unarmed**, so such a call is killed once, at the deadline, as before.
- **Local subcommands** get no trace file, no stall bound and no retry.
- **An operator's own `GIT_TRACE_PACKET` is overridden** for remote subcommands lyx runs; `doc.go` says so.
- The retry reruns the attempt only, inside the same network-gate hold, so a pause is waited on once.

## Check by hand

- Tests shorten `stallBound` and cover the stall, the answer-then-hang and the unarmed cases.
  They were not run against a real GitHub connection.
- The packet trace was checked to write a first line over HTTPS and local remotes.
  A git build that writes no packet trace for some transport, SSH included, would see every call on it stall twice and fail; worth a push and a fetch over SSH if any remote uses it.
- 30 s is chosen to be long enough for GitHub's first answer on a slow link; watch for spurious stall Warns on slow links.
