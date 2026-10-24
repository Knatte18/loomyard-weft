# Support log — git network calls get a stall bound and one retry

## Interview

A darn run holds no interview: the board entry is the whole statement, and every open question below is a self-pick.

## Rejected alternatives

- Lower the ten-minute deadline: a large clone or push legitimately runs for minutes, so a deadline short enough to catch a stall fails real transfers.
- Rely on `http.lowSpeedLimit` alone: it is already set, and curl applies it only after the connection is up, so a stalled TLS handshake escapes it.
- A per-subcommand deadline, short for ls-remote: it leaves push and fetch, one of the two seen stalls, unbounded by anything short.
- Pass the packet trace on an inherited file descriptor: Go's `ExtraFiles` is not supported on Windows, while a trace file path works everywhere.
- Poll the trace file throughout the call: one check at the stall bound answers the only question asked, whether the remote ever answered.
- Retry on a deadline kill too: after the remote answered, a push may have updated refs, so only a stall is known to be safe to retry.

## Review rounds

_No rounds yet._

## Question ledger

- Which sign marks a stall? Self-pick: an empty `GIT_TRACE_PACKET` file at the stall bound, checked against real HTTPS ls-remote and push, whose first trace line follows the server's answer.
- How long is the stall bound? Self-pick: 30 s.
- Which errors are retried? Self-pick: a stall only, once.
- How is a second stall reported? Self-pick: a `*GitError` with `Stalled` set and `Timeout` the stall bound, so `IsTransportFailure` reports it with no change.
