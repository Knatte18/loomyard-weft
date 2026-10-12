---
format: 5
approved: false
language: go
---

# Plan: run-time-by-role

`tools/tokencount` reports where each finished run's wall-clock time goes beside its tokens: each role's working time as turn time, the run's idle time, and a Producers table over the run's archived shed history, every share against the run's span.
The archived `seed.json` and `status.json` come from the run's newest `archive/<slug>/<sha>` tag in the weft repository, through `internal/shedrun`'s new tree-relative paths and exported seed decoder, so the tool spells neither the `shed` segment nor `_lyx`.

## Card Index

1 — shedrun-tree-paths — tree-relative seed and status paths and an exported seed decoder in shedrun
2 — transcript-working-time — turn-time intervals per transcript, unioned per role and per run
3 — archived-run-status — archive tags with commit dates, and the archived seed and status read from the newest tag
4 — run-timing — span, role times, idle and Producers rows from the intervals and the archived status
5 — report-time-columns — time and time share columns, the idle row, the Producers table and the All runs sums
6 — run-wiring — weft default whenever empty, foreign-hub refusal, in-flight bound, status read and the new title
7 — docs — package doc and launcher comments describe the time measurement

## Shared Decisions

### Decision: time-model

- **Decision:** Working time is the interval between consecutive timestamped user or assistant lines of one transcript unless the first ends the turn: an assistant line whose `stop_reason` is set and not `tool_use`, or a user line opening with the interrupt marker or with `<local-command-stdout>`.
- **Rationale:** Turn boundaries separate work from waiting without a threshold, decision record Decision 1.
- **Applies to:** cards 2, 4, 5, 7

### Decision: span-and-idle

- **Decision:** A run's span runs from the seed's `started_at`, else the earliest transcript line, bounded below by the next-older archive tag's commit time and above by the first history entry's end, to the latest history entry end; a run without archived status spans its transcript window and prints no Producers table. Idle is the span minus the union of the run's worktree transcripts' working intervals; orch transcripts never enter that union.
- **Rationale:** Decisions 3 and 4; the history's finish times are the measured fact on the board.
- **Applies to:** cards 4, 5, 6, 7

### Decision: errors-name-slug-tag-file

- **Decision:** A run id `ValidateRunID` refuses, a seed or status that fails to decode, and a timestamp that fails to parse abort the report with an error naming the slug, the archive tag and the file.
- **Rationale:** Decision 9; an archived record the tool cannot read is a fact to show, not to paper over.
- **Applies to:** cards 3, 4, 6

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
go test ./internal/shedrun ./tools/tokencount ./cmd/lyx
go test -tags integration ./internal/shedrun ./tools/tokencount ./cmd/lyx
