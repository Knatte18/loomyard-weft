---
format: 5
approved: false
language: go
---

# Plan: run-time-by-role

`tools/tokencount` reports where each finished run's wall-clock time goes: the time each role's sessions spend working, the time no session works, and the time between the run's shed rows, each as a share of the run's span.
The plan collects turn-time intervals per transcript, reads each run's archived shed seed and status from its newest archive tag in the weft through new `shedrun` helpers, computes the span, role times, idle and per-producer rows, prints them as new columns, an `idle` row and a Producers table, wires the weft default and the hub guard, and updates the docs.

## Card Index

1 — shedrun-tree-files — `shedrun` gains the git-tree path pair and a strict seed decoder over bytes
2 — tokencount-working-intervals — collect each transcript's turn-time intervals into role and run unions
3 — tokencount-shed-record — read a run's newest archive tag's seed and status from the weft
4 — tokencount-run-times — compute the span, role times, idle and Producers rows
5 — tokencount-time-report — print the time columns, the idle row, the Producers tables and the new title
6 — tokencount-shed-wiring — default the weft whenever empty, refuse a foreign hub, attach each run's record
7 — tokencount-time-docs — package doc and launcher comments describe the time measures

## Shared Decisions

- `cmd/lyx/testdata/test-budget.yaml` is raised in the card that adds each top-level test; the `tools/tokencount` row grows by the tests cards 2 to 6 add, and the `internal/shedrun` row by card 1's.
- Only `internal/shedrun` names the `shed` segment or decodes `Seed`; `tools/tokencount` spells neither file name and takes both paths from `shedrun.TreeFiles`.
- The weft is only read: tag listing runs through `gitexec.Run`, blobs through `gitrepo`'s `FileAtRevision`.
- Tests that run git carry the `integration` tag; every other new test is unit tier over synthetic transcripts and fakes.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./tools/tokencount ./internal/shedrun ./cmd/lyx
go test -tags integration ./tools/tokencount ./internal/shedrun ./cmd/lyx
