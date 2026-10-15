---
format: 5
approved: true
language: go
---

# Plan: sibling-overlap-check

A board write that gives a task a new `depends_on` entry another open task already waits on, where the two share a module label, still succeeds and its envelope's `notice` names the sibling, the shared dependency and the shared labels.
`boardengine` computes the siblings under the write lock and `boardcli` formats them; the `ly:board` skill tells every agent that sets `depends_on` to check siblings.

## Card Index

1 — engine-siblings — compute siblings under the write lock and return them from the depends_on-writing Board methods
2 — cli-sibling-notice — print the siblings in the write envelope's notice, document the warning and add the skill rule

## verify:

```bash
go build ./...
go vet -tags integration ./internal/boardengine/... ./internal/boardcli/... ./cmd/lyx/...
go vet -tags tmux ./internal/boardengine/... ./internal/boardcli/... ./cmd/lyx/...
go vet -tags llm ./internal/boardengine/... ./internal/boardcli/... ./cmd/lyx/...
go test ./internal/boardengine/... ./internal/boardcli/... ./cmd/lyx/...
go test -tags integration ./internal/boardengine/... ./internal/boardcli/... ./cmd/lyx/...
```
