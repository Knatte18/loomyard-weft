---
format: 5
approved: false
language: go
---

# Plan: run-window-bugs

Fix the seven defects bundled on board entry `run-window-bugs` so none recurs, and record the weft push finding, per `_lyx/discussion/decision-record.md` (D1–D7).
The cards run from the self-contained fixes (push-retry doc, stale-socket kit, status pane, `lyx loom fmt` and its directive line) to the shuttle start sequence (launch-carried skill load, then events-first readiness) and end with the task worktree's leftover-directory removal.

## Card Index

1 — push-retry-race-doc — name the weft main ref-update race in PushWithRetry's doc (D4)
2 — stale-socket-kit — move the datagram stale-socket helper into tmuxkit and use it in tmuxkit's dead-socket test (D5)
3 — statuspane-full-clear — clear screen and history on every redraw and wake the pane loop on resize (D3)
4 — loom-fmt-verb — add lyx loom fmt, formatting named Go files in place (D6)
5 — edit-directive-fmt-line — name lyx loom fmt in the edit directive (D6)
6 — launch-carried-load-turn — shuttle waits for a launch-carried skill-load turn instead of typing it (D2)
7 — claude-launch-skill-load — Claude carries the skill-load message on the launch line; PATTERN and docs reworded (D2)
8 — startup-events-readiness — start's startup probe answers ready from session start then turn start in the events (D1)
9 — leftover-worktree-removal — remove the task worktree directory git left behind, retrying a late-writer failure (D7)

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
