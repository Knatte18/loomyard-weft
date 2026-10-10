---
format: 5
approved: false
first_card: 10
language: go
---

# Plan: Discussion-Write on MultiLLMProducer: a chair writes, advisors answer

The Discussion-Write row has a second producer, `DiscussionSeats`, beside the unchanged single-agent `DiscussionWrite`: a chair that writes the discussion as the single writer does, plus one advisor seat per entry of `loom.yaml`'s `discussion_advisors` list, each studying the task and answering the chair over the session message channel, with `discussion_producer` selecting which of the two the row runs.
The retired generation built all of that, and its cards are in the code; this generation holds only what the pull request's review found wanting.
The one finding is the chair stencil's advisor protocol: it tells a chair with several advisors to give each an area of the design, where the operator's design sends every question batch, formed as the solo writer forms them, to every advisor and weighs all answers together, since each advisor may find a different weakness.
The decision record's D5 and D6 sentences about areas are superseded by that review and are not edited here; the record is read-only context for this round.
The decision record's carry-over entries were checked against the tree again: `discussion_advisors` is in `ConfigOpenMaps`, the gate-quiescence guard names `DiscussionSeats` as a writer engine, the entry test's chair result carries a passing gate outcome, and the orch trust-rule entry is design text no card touches.

## Card Index

10 — chair-batches-to-every-advisor — the chair stencil sends every question batch to every advisor and settles after weighing all answers, and the claims test pins it

## Shared Decisions

### Decision: chair-stencil-includes-the-discussion-stencil

- **Decision:** The chair's opening stencil, `loom-template-discussion-chair`, includes `loom-template-discussion` whole and `seat-directive-chair`, then adds its own advisors section; the discussion stencil's agent-facing body stays unchanged, so the chair's batches are the ones the included Step 3 describes, each question with its recommended answer and its alternatives.
  The rework edits only the advisors section, never the include or the included stencil.
- **Rationale:** the included stencil is the one source for the writer's batch shape, fence and gates, and the single-agent composer fills it with no include of its own; the finding is about how the chair distributes those batches, which lives in the chair's own section.
- **Applies to:** card 10

### Decision: claims-pin-the-protocol-both-ways

- **Decision:** the chair-stencil row of `wordingClaims` gains `must` claims for the new wording, every batch to every advisor and a settle after weighing all answers, and a `mustNot` claim for the retired area assignment, so the old protocol cannot return by a later edit.
- **Rationale:** the claims test is the stencils package's coverage of its prose contracts, and the retired wording was never pinned, which is how it reached the pull request.
- **Applies to:** card 10

### Decision: no-area-mode

- **Decision:** no operator-selectable area mode, no `loom.yaml` key and no `mode_rules` change: the Go-held autonomous sentence already sends the chair's design questions to the advisors named in its prompt and says nothing about areas, so it stands.
- **Rationale:** the review asks for the one replacement and names an area mode as a separate board note.
- **Applies to:** card 10

### Decision: pattern-overview-untouched

- **Decision:** no card edits `PATTERN.md`, since the task adds no cross-cutting invariant, so the parent's overview-cap decision needs no offset.
- **Rationale:** the change is stencil prose and its claims; every invariant it rides is already in the overview.
- **Applies to:** all cards

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
