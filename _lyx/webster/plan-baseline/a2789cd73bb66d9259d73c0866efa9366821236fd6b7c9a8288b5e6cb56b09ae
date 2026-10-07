---
format: 5
approved: true
language: go
---

# Plan: run-recovery — see what a run waits on, and bring a run back as it was

The operator and the orch can see what a run waits on, and a run comes back up as it was after a crash or a halt.
The Go-side waits (every `verifytree.Verify` site, shuttle's gate entries and its background-shell wait) show in `lyx loom status` with producer, elapsed time and detail, and on screen as a reed pane mark the status line renders.
A verify gate gets a timeout that kills the verify shell's process group on Unix.
Reed stops stacking a bare claude over a dormant orch and stops reviving an ungated shuttle strand whose run finished.
`lyx batten run` brings a crashed or halted child's driver back without resuming the child, and the orch resumes a child through `lyx loom resume`, a verb that only wakes a parked driver.

## Card Index

1 — reed-wait-mark-op — reed's one op that sets or clears a strand's pane wait mark, plus its PATTERN entry
2 — reed-status-line-wait-segment — the default status line renders marked panes with elapsed time through a `waits` token
3 — reed-unless-name-dormant — `--unless-name` skips the add over a dormant strand too
4 — reed-directory-retiring — reed's sessionless directory reports a retiring strand
5 — reed-resume-done-when — reed resume drops a non-live strand whose done-when paths all exist
6 — shuttle-done-when — shuttle sets an ungated run's output files as its strand's done-when list
7 — verifyrun-process-group — the verify shell runs in its own process group, killed whole on cancel or expiry
8 — verifytree-timeout — `verifytree.Verify` takes a timeout; an expiry is a failed verify, never an error
9 — verify-timeout-findings — each verify gate reports a timeout as a failed gate naming it
10 — verify-wait-note — loom status renders a running verify as producer, elapsed time and command
11 — shuttle-wait-marker — shuttle marks gate evaluation and the background-shell wait on disk and on screen
12 — loom-status-shuttle-wait — loom status reports shuttle's waits between the verify and parent-review notes
13 — landing-verify-wait-mark — the landing merge-in verify marks the driver strand through a told callback
14 — loom-resume-verb — `lyx loom resume` wakes a parked driver and refuses everything else with a way forward
15 — orch-stencil-resume — the orch role stencil names `lyx loom resume` for every halted-child resume
16 — batten-respawn-scoped — batten's spawn confirmation is per process, so a restarted batten re-spawns a driverless running child
17 — batten-halted-revive — batten revives a halted or awaiting child's dead driver once per episode, never resuming the child

## Shared Decisions

- Wait labels are one vocabulary across status and screen: `verify` in the status note and `verify <producer>` on screen for a verify, `gate <entry name>` for a shuttle gate entry, `background shells` for the shell wait.
- Every marker that `lyx loom status` reads carries a pid, and a dead pid reads as absent; the pane mark is display only and no Go decision reads it.
- A failure to set or clear a mark, or to write or remove a marker, is logged and changes no verdict, error return or gate outcome.
- Elapsed time is rendered by one formatter in `loomcli`, from an injectable clock; tmux computes the on-screen elapsed time itself.
- Every new reader of a child's driver strand (`lyx loom resume`, batten's revive) reads reed's sessionless `Directory`, never the seed's driver.

## verify:

```
go build ./...
go test ./...
go test -tags integration ./...
go test -tags tmux ./internal/reedengine/... ./internal/reedcli/... ./internal/loomcli/... ./internal/webstercli/...
go vet -tags llm ./...
```
