---
format: 5
approved: false
first_card: 10
language: go
---

# Plan: review loops converge from round 2, carrying their fresh findings

Review segments stop circling to their budget on fresh edge-case keys.
From round 2 on, the judge converges a segment once its latest round carries no BLOCKING finding and no MEDIUM-or-worse `design` finding on a key already open in an earlier round, and Go carries the open findings of every closing round into the decision record's `## Open risks`, where Plan-Write, rework and Describe each check them.
The template budget drops to three rounds, each review segment can set its own reviewer and fixer models, the fixer can start only after the review is accepted, and `lyx loom resume` resumes a recorded circling decision (issue #446).

The retired generation built all of that.
This generation fixes the rejected pull request's finding: the template's review models per segment, a parent decision added after the Discussion that no retired card picked up.

## Card Index

10 — template-review-models — template `review: opus[medium]` and `webster_review: sonnet[high]`, with the tests that pin the template

## Shared Decisions

### Decision: docs-without-doc-go

- **Decision:** `loomengine` carries no `doc.go`, so its docs are `config.go`'s header and `template.yaml`'s key comments; card 10 changes two template values whose key comments already describe them, so no doc text changes.
- **Rationale:** docs land in the card that changes the behavior, in the file that documents it.
- **Applies to:** card 10

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
