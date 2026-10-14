---
format: 5
approved: false
language: go
---

# Plan: run-window-bugs

Fix the seven defects bundled on board entry `run-window-bugs` and record the weft push finding, per the decision record's D1–D7: `start`'s startup probe reads readiness from events, the launch line carries the skill-load message, status panes clear screen and history and wake on a resize, the push retry's doc names the remote race, the stale-socket test helper moves into `tmuxkit`, `lyx loom fmt` formats named Go files and the edit directive names it, and a task worktree's leftover directory is finished with a bounded late-writer retry.

## Card Index

1 — push-retry-finding — record the weft push refusal finding in PushWithRetry's doc comment
2 — stale-socket-kit — move the datagram stale-socket helper into tmuxkit and use it in both packages
3 — statuspane-full-clear — redraw with a full screen and history clear, and wake the pane on a resize
4 — startup-ready-from-events — start's probe reads a session start then a turn start as ready
5 — launch-carried-load-turn — shuttle takes a launch-carried first skill-load turn, then color, retry, prompt
6 — claude-launch-skill-load — Claude puts the skill-load message on the launch line; PATTERN and docs reworded
7 — loom-fmt-verb — add lyx loom fmt, formatting named Go files in place
8 — edit-directive-fmt — name lyx loom fmt in the shared edit directive
9 — leftover-worktree-retry — finish a worktree directory git left unregistered, retrying a late writer

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
