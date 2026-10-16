# Creel stage 1 re-test fixes

All three batches recorded `done`; the scope guard raised only informational deviations.

## Batch 01-journal-tail-repair (head 3e74dff03)

Cards 1–3.
The journal tail repair cuts a torn last line with or without a trailing newline, sharing the fold's last-line rule.
A creel refusal whose stdout envelope cannot be written goes to stderr.
A stall reaches a waiter or an expired answer only from a reading no older than twice the probe ceiling.
Deviations: `internal/creelengine/deliver.go`, `internal/creelengine/deliver_test.go`, `internal/creelengine/journal_test.go`.

## Batch 04-spawn-exit-signal (head 393ad6b5c)

Cards 4–6.
The client's connect wait ends when its spawned daemon exits other than by a lost start race.
`log` lists each carried record as a `carried` event, after a fold and in a running daemon.
The sweep confirms a first miss after 10 s, caps its backoff at 50 s and never lengthens it on new mail.
Deviations: `internal/creelengine/client.go`, `internal/creelengine/journal_test.go`.
The fork reported two pre-existing flaky tests (a sweep row and a scheduler inject row), not caused by this batch.

## Batch 07-idle-probe-no-input-box (head 208a1c221)

Cards 7–11.
Shuttle's idle probe reports a pane with no input box, and creel names that stall.
A live receiver's probe wait is capped at its soonest inject's remaining TTL.
Shuttle gains an engine-neutral seam that accepts a provider's startup gate on a live strand.
The llm smoke strand gets its own window, accepts the startup gate and prints its pane on failure; the `llm` tier was not run, as no gate runs it.
Deviations: `internal/creelcli/smoke_helpers_test.go`, `internal/shuttleengine/seam_enforcement_test.go`.
