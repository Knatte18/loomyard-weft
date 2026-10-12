---
format: 5
approved: false
language: go
---

# Plan: run time by role in tokencount

`tools/tokencount` reports where each finished run's wall-clock time goes, beside its token use.
Per role and per run it measures working time as turn time, the union of a role's transcripts' working intervals clipped to the run's span, with sub-agents under `<role>+sub`; the `idle` row is the part of the span no session of the run is in a turn.
The span and a per-run Producers table come from the run's archived shed `status.json` and `seed.json`, read from its `archive/<slug>/<sha>` tag in the weft repository through `internal/shedrun`'s new tree-read pair and `gitrepo`'s read side.
The cards build bottom-up: the `shedrun` additions, the interval collection, the archive reader, the timing computation, the report, the command wiring, then the docs.

## Card Index

1 — shedrun-tree-reads — `shedrun` gains the run-id-relative `seed.json`/`status.json` path pair and a strict seed-bytes decoder
2 — working-intervals — `countFile` collects turn-bounded working intervals per role, with union and clip helpers
3 — shed-records — read the newest archive tag's seed and status from the weft, bounded for in-flight slugs, with decode errors naming slug, tag and file
4 — run-timing — compute the span, each role's time, idle and the Producers rows
5 — report-time-columns — `time` and `time share` columns, the `idle` row, the Producers table and the "All runs" sums
6 — main-wiring — the weft default for every run, the foreign hub refusal, per-slug shed reading and the new title
7 — docs-launchers — the package doc and the launcher comments

## Shared Decisions

- The `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` rises by each card's new top-level tests, from 12 to 18 across cards 2 to 6; cards 1 and 7 add rows and comments only.
- A tree read from git applies no run-id alias and no legacy `self` fallback: the run id is the slug.

## verify:

```
go build ./...
go vet -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags tmux ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags llm ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test ./internal/shedrun ./tools/tokencount
go test ./cmd/lyx -run TestTestBudget
go test -tags integration ./internal/shedrun ./tools/tokencount
```
