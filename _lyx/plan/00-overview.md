---
format: 5
approved: false
---

# Plan: Plan gaps: inserting a card, unreachable prescribed refusals

Close two gaps in how plans are written and reviewed (board task `plan-gaps`, issues #508 and #516).
Cards 1 and 2 add the plan-format renumber write path and its journal recovery to `internal/planparser`, Card 3 mounts them as `lyx loom insert-card`, `remove-card` and `restore-renumber` under webster's state-mutation lease with their refusal rows and docs, Card 4 names the verbs in the three stencils that edit plans, and Card 5 adds the paired writer rule and rubric bullet that a prescribed refusal must be reachable.

## Card Index

1 — planparser-renumber — insert or remove one card and renumber the later cards, journaled and reversed on a raw I/O failure
2 — planparser-restore — restore the plan directory from a renumber journal an interrupted call leaves
3 — loom-renumber-verbs — mount insert-card, remove-card and restore-renumber under webster's lease, with refusal rows and docs
4 — stencils-name-renumber-verbs — name the two verbs as the only way to insert or remove a card file
5 — prescribed-refusal-reachability — a prescribed refusal names its reaching input and test, writer rule and rubric bullet

## Shared Decisions

- Every refusal returns an error wrapping a `planparser` sentinel; `internal/loomcli` alone words the message and its `way forward:` clause, so the clause and its `refusal-spec.md` row are written in one card.
- Raw I/O failures are injected through `planparser.Renumber`'s `FS` seam, nil in production, never through a package-level variable, so the tests stay parallel.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/planparser/... ./internal/loomcli/... ./internal/websterengine/... ./contracts/stencils/... ./cmd/lyx/...
go vet -tags tmux ./internal/planparser/... ./internal/loomcli/... ./internal/websterengine/... ./contracts/stencils/... ./cmd/lyx/...
go vet -tags llm ./internal/planparser/... ./internal/loomcli/... ./internal/websterengine/... ./contracts/stencils/... ./cmd/lyx/...
go test ./internal/planparser/... ./internal/loomcli/... ./internal/websterengine/... ./contracts/stencils/... ./contracts/specs/... ./cmd/lyx/... ./internal/lyxcwd/...
go test -tags integration ./internal/planparser/... ./internal/loomcli/... ./internal/websterengine/... ./contracts/stencils/... ./contracts/specs/... ./cmd/lyx/... ./internal/lyxcwd/...
```
