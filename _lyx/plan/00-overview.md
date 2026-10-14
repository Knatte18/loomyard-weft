---
format: 5
approved: false
language: go
---

# Plan: run-window-bugs

Fix the seven defects bundled on board entry `run-window-bugs` so none recurs, and record the weft push finding.
Shuttle's `start` probe reads readiness from the events file and its launch line carries the skill-load message (D1, D2).
The shared status pane clears screen and history on every redraw and wakes on a resize (D3).
`PushWithRetry`'s doc names the weft ref-update race (D4).
The stale-socket test helper moves to `tmuxkit` (D5).
`lyx loom fmt` formats named Go files and the edit directive names it (D6).
The task worktree removal finishes a directory `git worktree remove` left refilled, retrying a late writer (D7).

## Card Index

1 — startup-events-readiness — start's startup probe is ready on a session start then a turn start in the events file, before any pane capture
2 — skill-load-on-launch-line — a skilled spec launches with the skill-load message, and shuttle awaits that turn, then types color, retry and prompt
3 — status-pane-full-clear — every status pane redraw clears screen and scrollback, and a terminal resize wakes the pane loop at once
4 — push-retry-race-finding — PushWithRetry's doc comment names the weft main ref-update race behind a retried remote-rejected refusal
5 — stale-socket-kit-helper — the datagram stale-socket helper moves to tmuxkit and tmuxkit's dead-socket test uses it with no wait
6 — loom-fmt-verb — lyx loom fmt rewrites named .go files in the worktree to their go/format form, refusing the whole call on a bad path
7 — edit-directive-fmt-line — the edit directive gains the line naming lyx loom fmt for a Go file's formatting
8 — leftover-worktree-dir-retry — removeWarpWorktreeDir removes the unregistered directory a failed git worktree remove left, retrying a late-writer error

## Shared Decisions

- Every new or reworded doc comment and Markdown line follows `PATTERN-comment-line-breaks` and semantic line breaks.
- `PATTERN.md` sits within a few bytes of `lyx loom lint-pattern`'s headroom line, so card 2's entry rewording keeps the entry's byte length and no other card grows `PATTERN.md`.
- New top-level tests raise their package's row in `cmd/lyx/testdata/test-budget.yaml` on the card that adds them (cards 6 and 8); every other card adds rows or subtests to existing tests.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
