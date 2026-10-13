---
format: 5
approved: false
language: go
---

# Plan: run time by role in tokencount

`tools/tokencount` gains where each finished run's wall-clock time goes: per-role turn time and time share beside the tokens, an `idle` row per role table, and a Producers table splitting the run's span by shed history entry, each run's span set from its archived `status.json` and `seed.json` read at its newest `archive/<slug>/<sha>` tag in the weft.
`internal/shedrun` gains the tree-relative paths and seed decoder that reading keeps inside `PATTERN-shed-run-directory`.

## Card Index

1 — shedrun-tree-paths — run-id-relative seed and status paths and a seed-bytes decoder in shedrun
2 — turn-intervals — collect each transcript's turn-time intervals and compute role time and idle over the transcript window
3 — archived-status-span — read the archived status and seed, set the span from them and split it into producer times
4 — time-columns — render time and time share columns, the idle row, the Producers tables and their All runs sums
5 — read-status-per-run — wire the status read into run, the weft default and its bounds, and the new report title
6 — describe-time — package doc and launcher comments describe the time figures

## Shared Decisions

- Durations print as `time.Duration` rounded to the second; time shares divide by the run's span, or the sum of spans in "All runs".
- The orch never enters a run's working-time union: `CountRun` reads only the run's worktree sessions, and `CountOrch` leaves `RunTally.Work` alone.

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
go vet -tags integration ./internal/shedrun ./tools/tokencount
go vet -tags tmux ./internal/shedrun ./tools/tokencount
go vet -tags llm ./internal/shedrun ./tools/tokencount
go test ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
```
