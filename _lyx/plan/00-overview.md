---
format: 5
approved: false
language: go
---

# Plan: run-time-by-role

`tools/tokencount` learns where each finished run's wall-clock time goes: per-role working time and time share beside the token columns, an `idle` row for the part of the run's span in which no session is in a turn, and a per-run Producers table built from the run's archived shed `status.json`.
Working time is turn time, taken from the transcripts; the span and the Producers rows come from the history of the status and the seed read from the run's `archive/<slug>/<sha>` tag in the weft repository.
`internal/shedrun` gains the tree-relative paths and the byte decoder that read needs, and `internal/shedengine`'s status shape does not change.

## Card Index

1 — shedrun-archived-reads — `shedrun` gains a run-id-relative path pair for `seed.json` and `status.json` and a byte decoder `ReadSeed` shares
2 — working-intervals — turn-based working intervals per transcript, unioned per role and per run
3 — archive-tags-and-status — archive tags with commit dates, and the archived seed and status read through `gitrepo`
4 — run-time-breakdown — the span, per-role time, idle and Producers rows of a run
5 — time-report — the `time` and `time share` columns, the `idle` row, the Producers tables and the notes
6 — wire-run-and-title — status read per run in `run`, the weft default, the `-hub` guard and the report title
7 — doc-and-launchers — the package doc and the launchers' description comments

## Shared Decisions

- A shed status is never read for a run in flight: `ReadShedRecord` returns no record when `<hub>/<slug>` exists.
- A run without a record, or with an empty history, uses its transcript window as its span; its section says no shed status was read and prints no Producers table.
- Orch transcripts never enter a run's `Work`, because `CountRun` reads only the run's own project directory.
- Design calls the record leaves open: a tree whose tag holds no `status.json` counts as no status read; a status with an empty history counts as no status read; the All-runs Producers shares divide by the summed spans of the runs that have a status; a sub-agent transcript's lines already contributed by an earlier transcript of the run are not counted again toward working time.

## verify:

```
go build ./...
go vet -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags tmux ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags llm ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
```
