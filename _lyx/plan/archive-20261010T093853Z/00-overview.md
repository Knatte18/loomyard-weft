---
format: 5
approved: false
language: go
---

# Plan: Discussion-Write on MultiLLMProducer: a chair writes, advisors answer

The Discussion-Write row moves from `SingleLLMProducer` onto `MultiLLMProducer`: the writer becomes the chair of a seat table built in Go, and `loom.yaml`'s new `discussion_advisors` list adds one advisor seat per entry, each studying the task and answering the chair over the session message channel.
The seat engine and the producer gain only additive seams, the config key and the two stencils land before the table builder that reads them, the fake seat runner learns to evaluate a gate before the recipe row switches to the table in one card, and the single-agent path is deleted once nothing references it, taking with it the composer that kept the chair stencil from including the chair seat directive.

## Card Index

1 — seat-table-optional-values — a seat table names the table values that may be empty, and exposes its composed prompts
2 — multillm-table-source-and-preparation — MultiLLMProducer takes a per-call table source and a fresh-spawn preparation
3 — loom-config-discussion-advisors — loom.yaml gains discussion_advisors, normalised and validated at load, carried whole through reconcile
4 — discussion-advisor-stencil — the advisor stencil ships, and the discussion stencil becomes the chair's
5 — discussion-table-builder — loomengine builds the Discussion-Write seat table from the location, config and registry
6 — seat-runner-gate-once — the fake seat runner evaluates a table's gate once on a done chair, as the fake shuttle does
7 — discussion-write-row-on-seats — the DiscussionWrite entry runs the table on MultiLLMProducer, with every Env filler following
8 — retire-discussion-spec — the single-agent DiscussionSpec, its composer and their tests are removed, and the chair stencil gains the chair seat directive
9 — docs-chair-and-advisors — the overview and the recipe spec describe the chair and its advisors

## Shared Decisions

### Decision: absent-key-takes-template

- **Decision:** `discussion_advisors` normalises null, `[]`, `""` and `[""]` to no advisors at load, and an absent key takes the template's one advisor, because `configengine.Load` fills a missing key from the template before the unmarshal, as it does for every loom key.
- **Rationale:** PATTERN-config-strictness makes the fill the loader's contract; the record's "absent key loads as zero advisors" holds for an in-memory `Config`, which is what the table builder's tests use.
- **Applies to:** cards 3, 5

### Decision: resolve-at-load-and-in-the-builder

- **Decision:** `LoadConfig` parses and resolves every advisor entry, loading the registry through `modelspec.LoadRegistry` from its own base directory only when the list holds entries, and refuses an empty, malformed or unknown one naming the key, the index and the way forward, as D4 requires; `DiscussionTable` resolves each entry again for the seat's model, effort and version, as `discussion` resolves there today.
- **Rationale:** D4 makes an unresolvable entry a config refusal at load, and `LoadConfig` already reads `burler.yaml` from the same base directory to validate the fan keys; the builder takes a `Config` that need not come through `LoadConfig`, so it keeps `Resolve`'s error.
- **Applies to:** cards 3, 5

### Decision: no-parent-name-parameter

- **Decision:** `DiscussionTable` takes no parent name; the seat engine renders every seat's parent directive from `Geometry.ParentName`, which `wire` already tells it.
- **Rationale:** a parameter the builder cannot use would be dead.
- **Applies to:** cards 5, 7

### Decision: mode-rules-under-a-new-name

- **Decision:** the advisor-aware mode text is `discussionModeRules` in the builder's file; `modeRules` stays untouched until `prompt.go` is deleted whole in card 8.
- **Rationale:** re-signing `modeRules` in place would leave `prompt.go` holding one function and a stale header, and a move would add a card for no behaviour.
- **Applies to:** cards 5, 8

### Decision: fake-seat-runner-evaluates-the-gate-once

- **Decision:** `shedfake.SeatRunner` evaluates a table's gate once on a done chair whose answered result carries no `GateOutcome`, as `shedfake.Shuttle` does for a gated run, and lands in its own card before the row switch.
- **Rationale:** the loomrecipe sequence and resume tests rely on Discussion-Write's own gate failing or passing inside the fake; a `RunFn` that evaluates the gate itself, as the parent-review test's does, returns its outcome, so the gate is never evaluated twice.
- **Applies to:** cards 6, 7

### Decision: chair-include-with-the-composer-removal

- **Decision:** the chair stencil's `{{template "seat-directive-chair"}}` include lands in card 8, with the deletion of `loomengine.composePrompt`, and card 4 adds only marker-free chair prose.
- **Rationale:** `composePrompt` fills the chair stencil through `stencil.FillOptional`, which refuses an include it has no entry for and never fills the block's markers, so the include cannot land while the single-agent composer's tests run.
- **Applies to:** cards 4, 5, 8

### Decision: test-budget-trajectory

- **Decision:** the `internal/loomengine` row of `cmd/lyx/testdata/test-budget.yaml` rises by two in card 5, and card 8 sets it to the count after the five deletions; card 3 adds rows only.
- **Rationale:** the builder's tests land before the tests they replace can go, since the recipe row still calls `DiscussionSpec` until card 7.
- **Applies to:** cards 5, 8

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
