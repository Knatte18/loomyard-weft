---
format: 5
approved: false
language: go
---

# Plan: run-time-by-role

`tools/tokencount` gains where each finished run's wall-clock time goes: per-role working time and time share beside the token columns, an `idle` row per role table, and a Producers table built from the run's archived shed `status.json` history, each per run and summed in "All runs".
`internal/shedrun` gains the tree-relative path pair and seed decoder a status read from a git tag needs, and the report reads each run's newest `archive/<slug>/<sha>` tag from the records repository.

## Card Index

1 — shedrun-tree-paths — tree-relative seed and status paths and DecodeSeed in shedrun
2 — working-intervals — collect each transcript's turn-time working intervals in countFile
3 — archived-span — read the archived seed and status and compute span, role times, idle and producers
4 — time-columns — print time columns, idle rows, Producers tables and the time rule
5 — run-wiring — default the records repository, refuse a foreign hub, skip in-flight slugs, read status per run, retitle
6 — package-doc — package doc and launcher comments describe the time report

## Shared Decisions

- Working intervals are collected per role in `RunTally.Working`, the same keys as `RunTally.Roles`; every time figure is `covered` of those intervals inside a window, so a role's time is a union and never a sum.
- `CountRun` ends with the transcript-window span; `run` replaces it from an archive only for a slug that is not in flight and has a tag.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags tmux ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags llm ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
```
