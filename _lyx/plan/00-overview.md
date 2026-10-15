---
format: 5
approved: false
language: go
---

# Plan: kickstart-per-card

Add a third kick-start pack layout to the Webster implementer fork, the per-card layout, behind a new `webster.yaml` switch `kickstart_pack_per_card` that defaults to on.
Under it the fork reads every card file of its batch first as the batch overview, and before each card's first edit it fetches that card's pack with a new read-only verb, `lyx loom card-pack <card>`, resolved against the tree as it stands at that card's start.
The file and inline layouts, the pack-off prompt and the recovery prompt keep their bytes; the begin-batch trace line records `pack_layout=card`, each `card-pack` call logs a `webster: card pack` line, and the fork read audit learns the new arm.

## Card Index

1 — per-card-switch — add the kickstart_pack_per_card webster.yaml key, default true
2 — card-pack-verb — add lyx loom card-pack, rendering one card's pack against the current tree
3 — per-card-layout — render the per-card layout in the fork prompt and pick it at begin-batch
4 — forkaudit-card-arm — teach tools/forkaudit the card arm from card-pack results and card pack trace lines

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
