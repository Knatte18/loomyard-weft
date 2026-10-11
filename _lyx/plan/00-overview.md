---
format: 5
approved: false
language: go
---

# Plan: run time by role in tokencount

`tools/tokencount` gains where each finished run's wall-clock time goes: each role's turn time and time share beside its tokens, an idle row of time no session of the run is in a turn, and a Producers table splitting the run's span by shed history entry.
The span comes from the run's archived shed `status.json` and `seed.json`, read from its `archive/<slug>/<sha>` tag in the weft through `gitrepo`'s read side, with paths and the seed decoder from `internal/shedrun`.

## Card Index

1 — shedrun-tree-record-paths — `RunFilesRel` and `DecodeSeed` let a git-tree reader address and decode a run's records
2 — turn-time-by-role — working intervals from turn boundaries, per-role union, idle, the time columns and the new title
3 — archived-shed-status — archive tags with dates, `ReadArchive` spans each run from its archived status, the weft default and the foreign `-hub` refusal
4 — producers-table — per-run and "All runs" Producers tables, the package doc paragraph and the launchers' comments

## Shared Decisions

- Every time figure is computed within its run and summed across runs for "All runs"; no union is taken across runs.
- The tool spells no `_lyx`, `shed` or `seed.json` literal: every path into a run's records comes from `shedrun.RunFilesRel`.
- Durations print as `time.Duration` rounded to the second, and every share prints `%.1f%%`.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
