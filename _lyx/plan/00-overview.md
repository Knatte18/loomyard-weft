---
format: 5
approved: false
language: go
---

# Plan: run time by role in tokencount

`tools/tokencount` gains wall-clock time beside tokens: each role's working time, measured as the union of its sessions' turns, each run's idle time, and a Producers table splitting the run's span by shed history entry.
The span comes from the run's archived `status.json` and `seed.json`, read from its newest archive tag in the weft repository through `internal/shedrun`'s new tree paths and seed decoder, and falls back to the transcript window when no status is archived.

## Card Index

1 — shedrun-tree-records — shedrun names a run's seed and status in a git tree and decodes seed bytes
2 — working-time — per-role working time, per-run idle and the time columns, over the transcript window
3 — archived-shed-status — archive tags with commit times, the archived shed record per run, the weft default and the -hub refusal
4 — span-and-producers — the span from the shed status, the Producers tables and the report title

## Shared Decisions

- Every new production member of `tools/tokencount` for time lives in `timing.go` (cards 2 and 4), and the archived-record reading in `shedstatus.go` (card 3).
- Durations print as `time.Duration` rounded to the second; shares print as `%.1f%%` and are 0 over a zero span.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags tmux ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags llm ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
```
