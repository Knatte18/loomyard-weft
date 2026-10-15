---
format: 5
approved: true
language: go
---

# Plan: orch context line in the batten overview

The prime's overview window, drawn by `lyx batten overview` in the `runs` strand, gains one first line for the orch session: its latest context reading against the soft and hard thresholds, the watcher's cycle phase and the cycle count, with `watcher down` once the watcher has exited.
`internal/orchengine` gets a lockless state reader so the pane never creates the lock file or blocks the watcher, and `internal/battencli` polls it per frame with the run lines' two-strike tolerance and draws the line from thresholds loaded once at start.

## Card Index

1 — orch-state-peek — `orchengine.PeekState` reads the orch state file with no lock and no mkdir
2 — overview-orch-line — the overview polls the orch state and draws the orch line above the run lines

## Shared Decisions

- The run lines render as the parent renders them at Plan time; `run-pane-timing` rebases on whichever lands first.
- No change to `lyx orch status`, the watcher, the orch state's fields or the orch config.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
