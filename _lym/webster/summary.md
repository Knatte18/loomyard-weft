# Creel mail delivered once

Both batches recorded `done`, with no departures and no renames.

## Batch 01-hook-output-seam

Head `28a7986793d892ef4333d78af4c79d15fe565fa8`.
The batch covers the boundary block replacement with additional context and the system message, the waiter hook's rewake labels, the release of unjournaled ids when an acknowledgement's journal write fails, and the daemon claiming mail into a marked waiter's ring.
Deviations (files changed outside the batch's targets): `internal/creelcli/hookread.go`, `internal/creelengine/server_test.go`, `internal/creelengine/verbs.go`, `internal/shuttleengine/engine.go`.

## Batch 05-client-ring-ack

Head `0bbcb6b4c9145d6b165e94303a58520a9c2e76ee`.
The batch covers the Go client's handling of a ring that names an unsent ack, hook-wait handing the ring's lines over as the wake, the ring echo, hook-read rendering the echo, the daemon's claim and bell log lines, and the tier3 smoke checks.
Deviations (files changed outside the batch's targets): `internal/creelcli/cli_test.go`, `internal/creelcli/hookread_test.go`, `internal/creelcli/smoke_llm_test.go`, `internal/creelengine/deliver_test.go`, `internal/creelengine/server_test.go`, `internal/creelengine/verbs.go`.

## Scope guard

The deviations above are the scope guard's full result.

## Verify gate findings

The plan-level verify failed on the package `github.com/Knatte18/loomyard/cmd/lym`, and no recorded card touched a failing package.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `829dfcf9ee6725466e3ede38b58b7054a7570413` fix: shorten creel post Long under the 2000-byte cap
