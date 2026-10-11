---
format: 5
approved: false
language: go
---

# Plan: run time by role in tokencount

`tools/tokencount` gains a time view of each finished run: each role's working time and share of the run's span beside its tokens, an `idle` row for the time no session of the run is in a turn, and a Producers table splitting the span by shed history entry.
Working time comes from turn boundaries in the transcripts; the span and the Producers split come from the run's shed `status.json` and `seed.json`, read from its `archive/<slug>/<sha>` tag in the weft repository through `gitrepo`'s read side, with paths and the seed decoder owned by `internal/shedrun`.

## Card Index

1 — shedrun-tree-records — `shedrun` gains tree-relative seed and status paths and the exported strict seed decoder
2 — working-time — each role's union of working intervals, from turn boundaries in its transcripts
3 — archived-status — archive tags with commit dates, and each run's span and producer split from its archived status
4 — time-report — time and time share columns, the idle row and the Producers tables in the report
5 — wire-run-time — every run reads its archived status, `-hub` is bounded, the doc and launchers describe time

## Shared Decisions

- Intervals are half-open `[Start, End)` in `time.Time`; durations print as `time.Duration` rounded to the second.
- No new symbol in `tools/tokencount` spells `_lyx`, the `shed` segment or `seed.json` as a path; `shedrun` does.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags tmux ./internal/shedrun ./tools/tokencount ./cmd/lyx
go vet -tags llm ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
```
