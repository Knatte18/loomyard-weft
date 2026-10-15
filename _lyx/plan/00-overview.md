---
format: 5
approved: false
first_card: 4
language: go
---

# Plan: Twill fork0, strict critical fixer, branch-coverage lens

Build the Twill design the benches settled for Plan-Write, on the code `overview-seats` left (`plan_lenses` and `discussion_lenses` fan keys, seat tables).
Cards 1 to 3 built fork0 at Plan-Write, the host-neutral fork boilerplate and re-prompt rule, the critical-fixer judge and the `twill-plan` lens changes.
They put the critical fixer in the shared directive for both hosts, which the decision record's last entry ("Critical fixer only at Plan-Write") supersedes: the critical fixer applies at Plan-Write only, and Discussion-Write keeps today's judge steps.
This round makes the judge a host-chosen part, so Plan-Write renders the critical fixer and Discussion-Write `main`'s judge steps, while both hosts keep the D3 re-prompt rule and the D5 boilerplate.

## Card Index

4 — judge-parts — the critical and plain judge steps as two part stencils, the critical one for Plan-Write alone

## Shared Decisions

### Decision: composed parts, no conditionals

The writer-fork and relaunch parts, and now the two judge parts, are stencils `Directive` fills itself and passes into directive markers, exactly one part of each pair non-empty, so every stencil keeps strict top-level markers and no `{{if}}`.
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
