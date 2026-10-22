---
format: 5
approved: false
language: go
---

# Plan: orch-strand-cycle

The orch lifecycle holds up when a session, its watcher or the machine goes away.
Reed reads a strand whose engine exited to its shell as dead, names a pane moved to another session, and lets an add reclaim a dead strand's name; creel never retypes a pasted inject and skips a torn-down worktree; the watcher gates the note without a reply, serializes the cycle request with a new `lyx orch withdraw`, discounts hub pause time, and withdraws a still-queued `/compact`; `lyx orch status` restarts a dead watcher and shows the pending request and a pane's placement.
The cards follow the decision record's D1 to D7 in dependency order: the pane listing fields and the age probe come before the liveness predicate that reads them, and reed's `StrandStatus.Elsewhere` before the orch status that shows it.

## Card Index

1 — reed-pane-command-fields — Both pane listings carry the foreground and start commands in a separator-joined tail field
2 — reed-pane-age-probe — Reed reads a pane's first-process age from /proc with the cgo clock tick rate
3 — reed-engine-liveness — One engine liveness predicate reads a shell-only pane past the 30s grace as dead at every strand reader
4 — conflict-stop-shell-only — Landing's conflict-session stop removes a present shell-only conflict strand
5 — reed-pane-elsewhere — A pane found in another session of reed's server is named on the status row and keeps its binding
6 — reed-reclaim-dead-name — A reed add removes a dead, visible, unpaused holder of the exact asked-for name before naming
7 — creel-delivery-teardown — Post-paste send failures map to not-landed, and a torn-down worktree is skipped in the strand listing
8 — orch-request-marker-lock — Every writer and remover of the cycle request compares and removes under the orch state lock
9 — orch-request-pause-discount — Hub pause time is excluded from a request's stale age and the handoff timeout
10 — orch-note-gate-no-reply — The note gate passes at a turn end with the note written, and the stencils ask for no reply
11 — orch-compact-withdraw-queued — A compaction seen while /compact is still queued withdraws it and clears the resume-noticed mark
12 — orch-withdraw-and-request-status — lyx orch withdraw, cycle_request in status, and held_by_pause on distill and refresh
13 — orch-status-restarts-watcher — status restarts a dead watcher under start.lock, stop takes start.lock, and status shows strand_elsewhere

## Shared Decisions

- The engine liveness predicate is `engineAliveIDs` (card 3); every strand-liveness reader in reed takes it, and every pane-level check (Selvage health, pane options, title repair) keeps `aliveIDSet`.
- `StrandStatus.Present`, `OwnWindow` and `PanePID` stay pane-level, so shuttle's `strandRemovable` and creel's process-to-strand mapping are unchanged by D1.
- The orch state lock (`Paths.StateLockPath`) is the one lock every writer and remover of `cycle-request` holds (card 8); `lyx orch withdraw` (card 12) uses it through `WithdrawCycleRequest`.
- The hub orch rules section "Dead strands after a reboot (#799)" is removed by the orch with `lyx orch rules` after landing, never by a card, since a task's code change never writes Bolt.
