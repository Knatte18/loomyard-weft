---
format: 5
approved: false
language: go
---

# Plan: run-window-bugs

Fix the seven defects bundled on board entry `run-window-bugs` so none recurs, and record the weft push finding.
The cards follow the decision record's D1–D7: the push-retry finding (D4), the shared stale-socket helper (D5), the status pane's full clear and resize wake (D3), the `lyx loom fmt` verb (D6), the task worktree leftover removal (D7), the launch-carried skill-load turn (D2), and `start`'s events-first readiness (D1), which relies on D2's first turn.

## Card Index

1 — push-retry-finding — name the remote ref-update race in PushWithRetry's doc comment
2 — stale-socket-kit — move the datagram stale-socket helper into tmuxkit and use it in both socket tests
3 — status-pane-clear — clear screen and history on every redraw, and wake the pane loop on a resize
4 — loom-fmt-verb — add lyx loom fmt and name it in the edit directive
5 — worktree-leftover-retry — finish the directory git worktree remove left, retrying a late writer
6 — skill-load-on-launch-line — launch a skilled spec with the skill-load message as its first turn
7 — startup-ready-from-events — let start's probe read readiness from a session start then a turn start

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
