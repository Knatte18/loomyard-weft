---
format: 5
approved: false
language: go
---

# Plan: run time by role

`tools/tokencount` learns where each finished run's wall-clock time goes.
For each role it reports the time the role's sessions spend in a turn, beside its tokens.
It reports the run's idle time, when no session of the run is in a turn, and a Producers table splitting the run's span by shed history entry.
Each figure is a share of the run's span, read from the run's archived `status.json` and `seed.json` at its newest archive tag in the weft.
`internal/shedrun` gains the git-tree path pair and byte decoder that keep the tool off its directory segment and seed format.
The measurement, the rendering, the record reading and the wiring land as separate cards, each with its tests.

## Card Index

1 — shedrun-tree-records — `shedrun` names a run's seed and status paths in a git tree and decodes seed bytes strictly
2 — turn-intervals-and-run-time — transcripts yield turn intervals, and a run's span, role times, idle and Producers split are measured
3 — report-time-columns — role tables gain time, time share and an idle row, and runs gain span notes and Producers tables
4 — archive-tag-records — archive tags carry commit and time, and a run's newest tag gives its seed and status
5 — run-reads-archived-status — `run` reads every run's archived status, refuses a foreign hub without -weft, skips in-flight runs, and the docs follow

## Shared Decisions

- Time lives in `tools/tokencount/runtime.go`; `count.go` keeps transcript walking and rendering.
- Every shed record path the tool reads comes from `shedrun.TreeRecordRels`, in production and in tests, so no file outside `internal/shedrun` spells the `shed` segment or `seed.json` (`PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer`).
- Durations print as `time.Duration` rounded to the second; shares print `%.1f%%`, `0.0%` over a zero span.

## verify:

```
go build ./...
go vet -tags integration ./tools/tokencount ./internal/shedrun ./cmd/lyx
go vet -tags tmux ./tools/tokencount ./internal/shedrun ./cmd/lyx
go vet -tags llm ./tools/tokencount ./internal/shedrun ./cmd/lyx
go test ./tools/tokencount ./internal/shedrun ./cmd/lyx
go test -tags integration ./tools/tokencount ./internal/shedrun ./cmd/lyx
```
