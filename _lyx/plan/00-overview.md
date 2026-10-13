---
format: 5
approved: false
language: go
---

# Plan: run time by role in tokencount

`tools/tokencount` gains each finished run's wall-clock split: each role's working time measured as turn time, the run's idle time, and a Producers table built from the run's archived shed history, each as a share of the run's span, beside the existing token columns.
The archived `seed.json` and `status.json` are read from the run's newest `archive/<slug>/<sha>` tag in the weft through `gitrepo`'s read side, with paths and the seed decoder from `internal/shedrun`.

## Card Index

1 — shedrun-tree-records — `shedrun` gains a run-id-relative seed and status path pair for a git tree and an exported strict seed decoder
2 — turn-time-model — measure turn time per role, the run's span and idle, and the per-producer split from transcripts and a decoded shed record
3 — time-report — render the time columns, the idle row, the Producers tables and their rules in the report
4 — archived-shed-record — list archive tags with their commits and dates, and read a run's archived seed and status into a shed record
5 — wire-run-time — wire the record into every counted run, apply the weft default always with its foreign-hub refusal, and retitle the report
6 — time-docs — describe the time measures in the package doc and the launchers

## Shared Decisions

- No change to `internal/shedengine`'s status shape; the tool decodes `status.json` into `shedengine.Status` and resolves each entry's end itself.
- `tools/tokencount` spells no `_lyx`, `shed` or `seed.json` literal and decodes no `Seed`; both paths come from `shedrun.RunRecordRels` and the seed from `shedrun.DecodeSeed`.
- Durations print as `time.Duration` rounded to the second; a time share with a zero span prints `n/a`.

## verify:

```
go build ./...
go vet -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags tmux ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags llm ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
```
