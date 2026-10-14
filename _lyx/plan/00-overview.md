---
format: 5
approved: false
language: go
---

# Plan: run-window-bugs

Fix the seven defects bundled on board entry `run-window-bugs` and record the weft push finding: the push retry's cause in its doc comment, one shared stale-socket test helper in a kit, a status pane that clears screen and history on every redraw and wakes on a resize, a `lyx loom fmt` verb named in the edit directive, a bounded retry of the task worktree directory `git worktree remove` left behind, a startup probe for `start` that reads readiness from the events, and a launch line that carries the skill-load turn.

## Card Index

1 — push-retry-cause-doc — name the weft ref-update race in PushWithRetry's doc comment
2 — stale-socket-kit — move the datagram stale-socket helper into socketkit and use it in reedengine's and tmuxkit's socket tests
3 — status-pane-full-clear-resize — redraw with cursor home, erase screen and erase history, and let a resize wake the pane loop
4 — loom-fmt-verb — add lyx loom fmt and the edit directive line naming it
5 — worktree-leftover-retry — finish a task worktree directory git left behind, retrying a late writer twice
6 — startup-events-proof — start's probe answers ready on a session start followed by a turn start
7 — launch-carried-skill-load — launch with the skill-load message, then color, retry and prompt

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
GOOS=windows go vet ./internal/statuspane ./internal/fabricengine
go test ./...
go test -tags integration ./...
```
