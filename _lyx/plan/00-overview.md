---
format: 5
approved: false
language: go
---

# Plan: run-window-bugs

Fix the seven defects bundled on board entry `run-window-bugs` so none recurs, and record the weft push finding.
The cards follow the decision record's D1 to D7: a shared stale-socket test kit (D5), the push retry finding in `gitrepo` (D4), a status pane that clears screen and history on every redraw and wakes on a resize (D3), a `lyx loom fmt` verb and its edit-directive line (D6), `start`'s startup probe reading readiness from events (D1), the skill-load message on the launch line (D2), and a bounded retry of the task worktree directory `git worktree remove` left behind (D7).
D1 lands before D2: until D2, a launch with skills reaches the capture path as today, and after it every `start` launch carries a first turn for the events proof.

## Card Index

1 — stale-socket-kit — move the datagram stale-socket helper into a test kit and use it in tmuxkit's dead-socket test
2 — push-retry-finding — name the weft main ref-update race in PushWithRetry's doc comment
3 — status-pane-full-clear — clear screen and history on every status pane redraw, and wake the loop on a resize
4 — loom-fmt-verb — add lyx loom fmt, an in-process go/format rewrite of named Go files in the worktree
5 — edit-directive-fmt — name lyx loom fmt in the shared edit directive
6 — start-reads-events-readiness — let start's startup probe answer ready from a session start then a turn start in events
7 — launch-carries-skill-load — put the skill-load message on the launch line and type the color after the load turn
8 — finish-leftover-worktree-dir — finish a task worktree directory git left refilled, retrying a late-writer failure

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
GOOS=windows go vet ./internal/fabricengine/ ./internal/statuspane/
GOOS=windows go vet -tags integration ./internal/fabricengine/ ./internal/statuspane/
go test ./...
go test -tags integration ./...
