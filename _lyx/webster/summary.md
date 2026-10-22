# Orch strand cycle holds up when a session, watcher or machine goes away

All three batches reported `status: done`, no batch needed recovery, and no digest listed a `Pattern-Departure:` trailer.
The scope guard's `deviations` are informational and are listed per batch below.

## Batch 01-reed-pane-command-fields (head `ee0403b26cb1cd13ab128142c8a91defa2dab2c7`)

Cards 1 to 4.
Both pane listings carry the foreground and start commands in a separator-joined tail field.
Reed reads a pane's first-process age from `/proc` with the cgo clock tick rate.
One engine liveness predicate reads a shell-only pane past the 30s grace as dead at every strand reader.
Landing's conflict-session stop removes a present shell-only conflict strand.
Deviations: `internal/reedengine/apply.go`.

## Batch 05-reed-pane-elsewhere (head `8b195c463484d3e1304b075a2ee29cba9cfd4c00`)

Cards 5 to 8.
A pane found in another session of reed's server is named on the status row and keeps its binding.
A reed add removes a dead, visible, unpaused holder of the exact asked-for name before naming.
Post-paste send failures map to not-landed, and a torn-down worktree is skipped in the strand listing.
Every writer and remover of the cycle request compares and removes under the orch state lock.
Deviations: `internal/creelcli/delivery.go`, `internal/orchengine/state.go`, `internal/orchengine/watcher_compact_test.go`, `internal/reedengine/ownwindow.go`.

## Batch 09-orch-request-pause-discount (head `bee86e0c9a5201532c17df78bbdd72a9be09a45a`)

Cards 9 to 13.
Hub pause time is excluded from a request's stale age and the handoff timeout.
The note gate passes at a turn end with the note written, and the stencils ask for no reply.
A compaction seen while `/compact` is still queued withdraws it and clears the resume-noticed mark.
`lyx orch withdraw`, `cycle_request` in status and `held_by_pause` on distill and refresh are added.
`status` restarts a dead watcher under `start.lock`, `stop` takes `start.lock`, and `status` shows `strand_elsewhere`.
Deviations: `internal/orchcli/cli_test.go`, `internal/orchcli/status.go`, `internal/orchcli/stop.go`, `internal/orchengine/watcher.go`, `internal/orchengine/watcher_test.go`.
