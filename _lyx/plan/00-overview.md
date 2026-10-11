---
format: 5
approved: false
language: go
---

# Plan: run time by role in tokencount

`tools/tokencount` gains wall-clock time beside tokens for each finished run.
Each role's working time comes from its transcripts' turns.
An `idle` row holds the part of the run's span no session of the run works in, and a Producers table splits the span by shed history entry.
The span comes from the run's archived `status.json` and `seed.json`, read at its archive tag in the weft through `internal/shedrun` and `gitrepo`'s read side.

## Card Index

1 — shedrun-tree-reads — add the tree-relative seed and status paths and the seed byte decoder to shedrun
2 — turn-time-columns — measure turn time per role and print time, time share and idle in the role tables
3 — archived-run-span — read the archived status and seed into the run's span and print the Producers tables
4 — weft-default-and-hub-guard — read every run's span, default the weft whenever it is empty, refuse a foreign hub
5 — report-docs — describe the time model in the package doc and the launchers

## Shared Decisions

### Decision: one interval toolkit

- **Decision:** `tools/tokencount/runtime.go` (card 2) holds the interval type and its union, clip, sum and uncovered helpers, and card 3's Producers idle reuses them.
- **Rationale:** idle per run and idle per producer are the same arithmetic over the same working union.
- **Applies to:** cards 2 and 3

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
