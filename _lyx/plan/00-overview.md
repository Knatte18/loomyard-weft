---
format: 5
approved: false
language: go
---

# Plan: run time by role in tokencount

`tools/tokencount` gains where each finished run's wall-clock time goes: each role's turn time and time share beside its tokens, an `idle` row for the time no session of the run is in a turn, and a per-run Producers table splitting the run's span by shed history entry, with sums in "All runs".
The span comes from the run's archived `status.json` and `seed.json`, read from its newest `archive/<slug>/<sha>` tag in the weft through `internal/shedrun`'s new tree paths and seed decoder.

## Card Index

1 — shedrun-tree-reads — shedrun gains tree-relative seed and status paths and `DecodeSeed`
2 — turn-time — per-role turn time, time share and the idle row over the transcript window
3 — archived-status — archive tags with commit times, the archived status sets the span, and the Producers table
4 — wire-status — `run` reads every run's status from the records repository, refuses a foreign `-hub` without the repository flag, docs and launchers

## Shared Decisions

- Durations print as `time.Duration` rounded to the second, and shares as `%.1f%%`.
- Status and seed are read only through `shedrun`'s tree paths and `DecodeSeed`, so the tool spells no `_lyx`, `shed` or `seed.json` literal.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/shedrun/... ./tools/tokencount/... ./cmd/lyx/...
go vet -tags tmux ./internal/shedrun/... ./tools/tokencount/... ./cmd/lyx/...
go vet -tags llm ./internal/shedrun/... ./tools/tokencount/... ./cmd/lyx/...
go test ./internal/shedrun/... ./tools/tokencount/... ./cmd/lyx/...
go test -tags integration ./internal/shedrun/... ./tools/tokencount/... ./cmd/lyx/...
```
