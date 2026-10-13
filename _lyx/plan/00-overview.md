---
format: 5
approved: false
language: go
---

# Plan: run time by role in tokencount

`tools/tokencount` gains where each finished run's wall-clock time goes: each role's working time and time share beside its tokens, an `idle` row for the time no session of the run is in a turn, and a Producers table splitting the run's span by shed history entry.
Working time comes from transcript turns, the span from the run's archived `seed.json` and `status.json`, read from its `archive/<slug>/<sha>` tag in the weft through `shedrun`'s new git-tree read side.

## Card Index

1 — shedrun-tree-reads — shedrun gains run-id-relative seed and status paths and a strict seed decoder for a tree read from git
2 — turn-intervals — CountRun records each transcript's working intervals by role and for the whole run
3 — run-time — MeasureRunTime computes the span, role times, idle and producer times
4 — render-time — the role tables gain time columns and an idle row, and the report gains Producers tables and its new title
5 — archived-records — run reads each run's archived seed and status from the weft and refuses a foreign hub without -weft

## Shared Decisions

- Durations print as `time.Duration` rounded to the second; time shares print with one decimal and a `%`.
- The interval helpers and the time computation live in `tools/tokencount/span.go`.

## verify:

```
go build ./...
go vet -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags tmux ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags llm ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
```
