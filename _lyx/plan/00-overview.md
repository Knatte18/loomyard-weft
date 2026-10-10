---
format: 5
approved: false
language: go
---

# Plan: Discussion-Write on MultiLLMProducer: a chair writes, advisors answer

The Discussion-Write row moves from `SingleLLMProducer` onto `MultiLLMProducer`: the writer becomes the chair of a seat table built in Go, and `loom.yaml`'s new `discussion_advisors` list adds one advisor seat per entry, each studying the task and answering the chair over the session message channel.
The seat engine and the producer gain only additive seams, the config key and the two stencils land before the table builder that reads them, the recipe row switches to the table in one card, and the single-agent path is deleted once nothing references it.

## Card Index

1 — seat-table-optional-values — a seat table names the table values that may be empty, and exposes its composed prompts
2 — multillm-table-source-and-preparation — MultiLLMProducer takes a per-call table source and a fresh-spawn preparation
3 — loom-config-discussion-advisors — loom.yaml gains discussion_advisors, normalised and validated at load, carried whole through reconcile
4 — discussion-advisor-stencil — the advisor stencil ships, and the discussion stencil becomes the chair's
5 — discussion-table-builder — loomengine builds the Discussion-Write seat table from the location, config and registry
6 — discussion-write-row-on-seats — the DiscussionWrite entry runs the table on MultiLLMProducer, with every Env filler and fake following
7 — retire-discussion-spec — the single-agent DiscussionSpec, its composer and their tests are removed
8 — docs-chair-and-advisors — the overview and the recipe spec describe the chair and its advisors

## Shared Decisions

### Decision: absent-key-takes-template

- **Decision:** `discussion_advisors` normalises null, `[]`, `""` and `[""]` to no advisors at load, and an absent key takes the template's one advisor, because `configengine.Load` fills a missing key from the template before the unmarshal, as it does for every loom key.
- **Rationale:** PATTERN-config-strictness makes the fill the loader's contract; the record's "absent key loads as zero advisors" holds for an in-memory `Config`, which is what the table builder's tests use.
- **Applies to:** cards 3, 5

### Decision: resolve-in-the-builder

- **Decision:** `LoadConfig` parses every advisor entry and refuses an empty or malformed one naming the key, the index and the way forward; registry resolution of each entry happens in `DiscussionTable`, with the same naming, where `discussion` resolves today.
- **Rationale:** `LoadConfig` has no registry, and resolving at the Discussion-Write call keeps the builder's inputs the ones D2 names.
- **Applies to:** cards 3, 5

### Decision: no-parent-name-parameter

- **Decision:** `DiscussionTable` takes no parent name; the seat engine renders every seat's parent directive from `Geometry.ParentName`, which `wire` already tells it.
- **Rationale:** a parameter the builder cannot use would be dead.
- **Applies to:** cards 5, 6

### Decision: mode-rules-under-a-new-name

- **Decision:** the advisor-aware mode text is `discussionModeRules` in the builder's file; `modeRules` stays untouched until `prompt.go` is deleted whole in card 7.
- **Rationale:** re-signing `modeRules` in place would leave `prompt.go` holding one function and a stale header, and a move would add a card for no behaviour.
- **Applies to:** cards 5, 7

### Decision: fake-seat-runner-evaluates-the-gate-once

- **Decision:** `shedfake.SeatRunner` evaluates a table's gate once on a done chair, as `shedfake.Shuttle` does for a gated run.
- **Rationale:** the loomrecipe sequence and resume tests rely on Discussion-Write's own gate failing or passing inside the fake, and the parent-review tests on the gate's preparation and re-prompt shape.
- **Applies to:** card 6

### Decision: test-budget-trajectory

- **Decision:** the `internal/loomengine` row of `cmd/lyx/testdata/test-budget.yaml` rises by one in card 3 and by two in card 5, and card 7 sets it to the count after the five deletions.
- **Rationale:** the builder's tests land before the tests they replace can go, since the recipe row still calls `DiscussionSpec` until card 6.
- **Applies to:** cards 3, 5, 7

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
