---
format: 5
approved: true
language: go
---

# Plan: run-window-bugs

Fix the defects bundled on board entry `run-window-bugs` so none recurs, and record the weft push finding the entry asks for.
Shuttle's launch line carries the skill-load turn, and `start`'s startup probe reads readiness from the events before the pane.
The shared status pane clears screen and history on every redraw and wakes on a resize.
The weft push refusal's cause goes into the push retry's doc comment.
The stale-socket helper moves into `tmuxkit`.
A `lyx loom fmt` verb gives agents a sanctioned Go formatter, named in the edit directive.
A task worktree directory that `git worktree remove` deregistered but left refilled is finished with a bounded late-writer retry.

## Card Index

1 — skill-load-on-launch-line — carry the skill-load message on the launch line and await its turn before the color, retry and prompt
2 — startup-ready-from-events — let start's startup probe read a session start then a turn start from the events before the pane
3 — status-pane-full-clear — clear screen and history on every status pane redraw and wake the loop on a resize
4 — push-retry-race-doc — name the weft main ref-update race in PushWithRetry's doc comment
5 — stale-socket-kit-helper — move the datagram stale-socket helper into tmuxkit and use it in the dead-socket test
6 — loom-fmt-verb — add lyx loom fmt for named Go files and name it in the edit directive
7 — worktree-leftover-retry — finish a deregistered, refilled task worktree directory with a bounded late-writer retry

## Shared Decisions

- Cards 1 and 2 both edit `internal/shuttleengine/doc.go`: card 1 its skill-loading and color paragraphs, card 2 its startup paragraph.
- Cards 6 and 7 both edit `cmd/lyx/testdata/test-budget.yaml`: card 6 the `internal/loomcli` row, card 7 the `internal/fabricengine` row, each raised by one.
- Every new wait is injected: shuttle's load-turn poll runs on the run's clock, the status pane's wait on its `After` seam, and the leftover retry's on its `sleep` seam, per `PATTERN-test-speed`.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
