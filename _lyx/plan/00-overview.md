---
format: 5
approved: false
language: go
---

# Plan: run-time-by-role

`tools/tokencount` gains time beside tokens: each role's working time, measured as turn time from the run's transcripts, an `idle` row for the part of the run's span no session works in, and a per-run "Producers" table splitting the span by shed history entry, each summed in "All runs".
The span and the Producers table come from the run's archived `status.json` and `seed.json`, read through `internal/shedrun`'s new tree-relative paths and seed decoder from the run's newest `archive/<slug>/<sha>` tag in the weft.
A run without archived status falls back to its transcript window.

## Card Index

1 — shedrun-tree-records — tree-relative seed and status paths and a seed decoder over bytes in `internal/shedrun`
2 — turn-time — turn-time working intervals per role, `time`/`time share` columns and an `idle` row
3 — archived-span — read a run's newest archive tag's status and seed and derive its span
4 — producers-table — split the span by history entry into per-run and "All runs" Producers tables
5 — weft-wiring — read every run's archived status in `run`, bound the weft default, retitle the report
6 — time-docs — document the time columns, idle, span and Producers in the package doc and launchers

## Shared Decisions

### Decision: intervals-in-one-file

- **Decision:** Interval arithmetic (`Interval`, `union`, `clip`, `length`) and the turn rule live in `tools/tokencount/worktime.go`; the archive read, the span and the Producers split live in `tools/tokencount/archive.go`.
- **Rationale:** `count.go` already holds the token tally; the time logic is a separate concern its own tests address.
- **Applies to:** cards 2, 3, 4

### Decision: carry-over-finding-checked

- **Decision:** The Discussion-Review carry-over finding `named-inflight-slug-reads-older-archive` is covered by Decision 8's worktree-existence bound, which card 5 implements; it is not separate scope.
- **Rationale:** the package doc defines a run as finished once its pair is torn down, which removes the worktree, so an existing `<hub>/<slug>` marks the run in flight.
- **Applies to:** card 5

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
