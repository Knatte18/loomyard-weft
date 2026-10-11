---
format: 5
approved: false
---

# Plan: run time by role in `tools/tokencount`

`tools/tokencount` reports where each finished run's wall-clock time goes, beside the tokens it already counts.
The plan adds `shedrun`'s two reads for a tree taken from git, the archived `status.json` and `seed.json` read from the weft's `archive/<slug>/<sha>` tag, the per-role working time from turn boundaries, the run's span and idle remainder, the Producers split by shed history entry, the report columns and tables, the wiring in `run` with its `-hub` refusal, and the docs.
New production code in the tool lands in two new files, `archive.go` and `worktime.go`, so each card's diff stays local to one concern.

## Card Index

1 — shedrun-record-reads — `shedrun` gains the run-id-relative path pair for a run's records and a strict seed decoder over bytes
2 — archive-read — list the archive tags, pick a slug's newest, and read its status and seed from the weft, with the in-flight rule
3 — working-intervals — collect each transcript's turn-based working intervals and union them per role
4 — run-time — compute a run's span, role times and idle
5 — producer-time — split the span by shed history entry into per-producer entries, elapsed and idle
6 — time-report — print the time columns, the idle row and the Producers table, with the measurement statements
7 — time-docs — update the package doc and the launcher description comments for the time report
8 — archive-wiring — read the archived records for each run in `run`, refuse a foreign hub when the weft flag is empty, retitle the report

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
