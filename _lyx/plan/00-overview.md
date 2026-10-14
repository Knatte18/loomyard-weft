---
format: 5
approved: false
language: go
---

# Plan: run-window-bugs

Fix the seven defects bundled on board entry `run-window-bugs` so none recurs, and record the weft push finding the entry asks for.
The cards follow the decision record's D1 to D7: the status pane's full clear and resize wake, the stale-socket helper's move into a kit, the push-retry finding, the `lyx loom fmt` verb and its edit-directive line, `start`'s events-first startup probe, the launch-carried skill load, and the bounded retry of a task worktree directory `git worktree remove` left behind.

## Card Index

1 — statuspane-full-clear — the status pane redraw clears screen and history, and a resize wakes the loop at once (D3)
2 — stale-socket-kit — the datagram stale-socket helper moves into `tmuxkit`, and the dead-socket test uses it with no wait (D5)
3 — push-retry-finding — `PushWithRetry`'s doc comment names the remote ref-update race behind a retried weft push refusal (D4)
4 — loom-fmt-verb — `lyx loom fmt <file>...` rewrites named Go files to their `go/format` form, with its spec rows and docs (D6)
5 — edit-directive-fmt — the edit directive names `lyx loom fmt` as the way to fix a Go file's formatting (D6)
6 — startup-events-readiness — `start`'s startup probe reads a session start then a turn start in the events file as ready, ahead of the capture (D1)
7 — launch-line-skill-load — a spec naming skills launches with the skill-load message, and the color, retry and prompt follow its turn end (D2)
8 — worktree-leftover-retry — a task worktree removal finishes a directory `git worktree remove` left refilled, retrying a late-writer failure (D7)

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
