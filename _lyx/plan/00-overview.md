---
format: 5
approved: false
language: go
---

# Plan: plan gaps — inserting a card, unreachable prescribed refusals

Give agents a sanctioned way to insert or remove a plan card, through `lyx loom insert-card` and `lyx loom remove-card` backed by a journaled renumber writer in `internal/planparser` and a `lyx loom restore-renumber` recovery verb, bounded by webster's begun batches and its state-mutation lease.
Then tell the plan writers and Plan-Review about the verbs, and add a writer rule and a matching review bullet that a prescribed refusal must be reachable.

## Card Index

1 — last-begun-card — websterengine reports the last card of the last begun batch from the run record
2 — plan-renumber — planparser inserts, removes and restores plan cards through a journaled renumber writer
3 — renumber-verbs — lyx loom insert-card, remove-card and restore-renumber with their refusals and docs
4 — stencil-rules — plan, rework and Plan-Review stencils name the verbs and the reachability rule

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
