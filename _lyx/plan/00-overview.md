---
format: 5
approved: false
language: go
---

# Plan: board-lock-running

A board entry held by a run stays the scope that run executes.
`boardengine` declares the run-status form once and refuses, inside its write critical section, any write whose net change on a run-held entry touches more than `status` or removes it, naming the run and the ways forward.
Loom composes its per-transition status through the same form, and `lyx batten run` claims the entry with `running · Worktree-Create` before it creates the pair, so the lock starts at the claim.

## Card Index

1 — run-lock — declare the run-status form and refuse scope changes to a run-held entry in the board's critical section
2 — shared-hub-board — one hub-board opener and one absent-or-done-skipping run-status write, used by loom
3 — batten-claim — batten's Worktree-Create row claims the entry before creating the pair

## Shared Decisions

- The lock is checked once, in `Board.boardCriticalSection` after the mutation and before the save, against the entries as loaded under the board file lock; no store mutator carries its own copy, so batch, merge and prune are judged on their combined net change.
- `RunStatus` and `IsRunStatus` are the only spelling of the `<state> · <producer>` form; loom's `boardStatus` and batten's claim compose through `RunStatus`, and the lock recognizes through `IsRunStatus`.
- Decision 10 holds: no PATTERN entry; the lock is documented in `internal/boardengine/doc.go`.
- The carry-over findings `prune-collides-with-locked-field-rule` and `unlock-of-live-run-unbounded` are checked against the code: Card 1's prune exception and its refusal text cover them, and neither is planned as scope of its own.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
