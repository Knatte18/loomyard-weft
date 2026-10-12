---
format: 5
approved: true
language: go
---

# Plan: Webster pack-on fork reads all first

With a kick-start pack, Webster's implementer fork is told to read every card file of its batch and every span the pack lists under every card in its first turn, in parallel, before any edit, then the files no span covers and the unresolved glyphs' files, still before any edit.
The rule that a file an earlier card of the batch edited is read whole in that card's step 1, its member found by name, goes away; line numbers obtained before the first edit hold before any edit only, and an edited file changes only through the Edit tool.
Only the pack-on stencil texts, their wording pins and `internal/websterengine/doc.go` change; pack-off, recovery and verify-fix prompts render byte-identically.

## Card Index

1 — pack-on-read-all-first — rewrite the pack-on FRESH-READ, card-read and locate fragments for a first-turn read of every card and span, with their wording pins and doc.go

## verify:

```bash
go build ./...
go vet -tags integration ./contracts/stencils/... ./internal/websterengine/...
go vet -tags tmux ./contracts/stencils/... ./internal/websterengine/...
go vet -tags llm ./contracts/stencils/... ./internal/websterengine/...
go test ./contracts/stencils/... ./internal/websterengine/...
go test -tags integration ./contracts/stencils/... ./internal/websterengine/...
```
