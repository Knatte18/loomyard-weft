---
format: 5
approved: false
language: go
---

# Plan: board-field-drop

A board.json rewrite by a `lyx` binary keeps every entry key and top-level key that binary does not know, so a deploy that adds a board field stops losing its data to a long-lived run from an older binary that writes status.
The `lyx cleanup` PR's open sweep race is closed: `internal/lyxtemp`'s removal retry is limited to the lock-file recreation symptom and bounded by a per-sweep time budget.

## Card Index

1 — board-unknown-keys — boardengine keeps unknown entry and top-level keys as raw JSON across every write path, with the retired-key rule in doc.go
2 — board-cli-kept-keys — pin that `get`, `list-full` and the upsert, promote and merge envelopes print kept keys, and say so in boardcli's doc.go
3 — sweep-retry-budget — lyxtemp retries only the not-empty removal symptom, under one injected-clock budget per sweep

## Shared Decisions

- Kept keys live on `Task.Unknown` and the store file's `Unknown` as `json.RawMessage`, and nothing decodes them into a Go type.
- No new top-level test: the board rows extend `TestLoad`, `TestRunLock` and `TestCLI`'s `stepLookupContract`, and the sweep rows extend `TestSweep`.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/boardengine/... ./internal/boardcli/... ./internal/lyxtemp/...
go vet -tags tmux ./internal/boardengine/... ./internal/boardcli/... ./internal/lyxtemp/...
go vet -tags llm ./internal/boardengine/... ./internal/boardcli/... ./internal/lyxtemp/...
go test ./internal/boardengine/... ./internal/boardcli/... ./internal/lyxtemp/...
go test -tags integration ./internal/boardengine/... ./internal/boardcli/... ./internal/lyxtemp/...
```
