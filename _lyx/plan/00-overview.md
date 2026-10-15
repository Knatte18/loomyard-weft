---
format: 5
approved: true
language: go
---

# Plan: Twill fork0, strict critical fixer, branch-coverage lens

Build the Twill design the benches settled for Plan-Write, on the code `overview-seats` left (`plan_lenses` and `discussion_lenses` fan keys, seat tables).
The twill directive's judge becomes a critical fixer at both hosts, and its fork boilerplate and re-prompt rule become host-neutral.
The directive gains an optional writer-fork part, rendered from its own stencil through `internal/twilldirective`, that Plan-Write alone asks for: the lead explores, one fork (fork0) writes the plan, and the lead reads the finished plan, spawns the lens forks and judges their findings.
The `twill-plan` fan gains `branch-coverage`, and `forced-edits` and `criteria-coverage` each gain one sentence.

## Card Index

1 — critical-judge — the critical-fixer judge, host-neutral boilerplate and re-prompt rule in the shared directive, for both hosts
2 — writer-fork — the optional fork0 part and relaunch stencils, switched on for Plan-Write alone
3 — plan-lenses — the branch-coverage lens and the registration and operand sentences in the twill-plan fan

## Shared Decisions

### Decision: composed parts, no conditionals

The writer-fork part and the relaunch paragraphs are two stencils `Directive` fills itself and passes into two optional directive markers, exactly one non-empty, so every stencil keeps strict top-level markers and no `{{if}}`.
A host value left empty fails the part's fill with an error naming its marker.

### Decision: gate unchanged

The Plan-Write row's gate, outputs and fork authorization do not change; `lens-review-missing` stays pinned by `internal/loomshed`'s gate test and `internal/shedrecipe`'s Plan-Write entry test, which no card touches.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
