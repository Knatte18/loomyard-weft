---
format: 5
approved: false
---

# Plan: run time by role

`tools/tokencount` gains where each finished run's wall-clock time goes: per-role working time and time share beside the token columns, an `idle` row for the span no session of the run is in a turn, and a per-run Producers table built from the run's archived shed `status.json` history, read from its `archive/<slug>/<sha>` tag in the weft.
`internal/shedrun` gains the run-id-relative record paths and seed decoder the tool reads a git tree with.

## Card Index

1 — shedrun-archived-records — run-id-relative seed and status paths and a seed decoder over bytes in shedrun
2 — role-working-time — per-role turn time, idle and the time columns, over the transcript window
3 — archived-run-span — the span from the archived seed and status, the -hub refusal and the in-flight bound
4 — producers-table — the Producers table per run and in All runs
5 — tokencount-docs — package doc and launcher comments for time, idle and Producers

## Shared Decisions

- Times are `time.Duration` rounded to the second; shares are `%.1f%%` of the table's span.
- No union is taken across runs: "All runs" sums each run's own figures.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
