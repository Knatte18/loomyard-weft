---
format: 5
approved: false
language: go
---

# Plan: run-time-by-role

`tools/tokencount` gains where each finished run's wall-clock time goes: each role's working time and time share beside its tokens, an `idle` row for the time no session of the run is in a turn, and a Producers table splitting the run's span by shed history entry.
Working time is turn time read from the transcripts the tool already walks; the span comes from the run's archived shed `status.json` and `seed.json`, read from its archive tag in the weft through `gitrepo`'s read side and decoded through `internal/shedrun`.

## Card Index

1 — shedrun-tree-records — tree-relative seed and status paths and an exported seed decoder in `internal/shedrun`
2 — role-working-time — working time per role and idle per run as role-table columns and an `idle` row, spanning the transcript window
3 — archived-run-span — read each run's archived status and seed from its archive tag and span the run from them, with the -weft default and the -hub refusal
4 — producers-table — per-run and summed Producers tables from the shed history, and the launchers' descriptions

## Shared Decisions

- Durations print as `time.Duration` rounded to the second; time shares divide by the table's span, the sum of the runs' spans in "All runs".
- "All runs" sums per-run figures and takes no union across runs.
- `tools/tokencount` names no `_lyx` literal, `shed` segment or `seed.json`: it reaches the archived records only through `shedrun.TreeSeedRel`, `shedrun.TreeStatusRel` and `shedrun.DecodeSeed`.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
