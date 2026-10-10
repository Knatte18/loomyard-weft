---
format: 5
approved: true
language: go
---

# Plan: Discussion-Write on MultiLLMProducer: a chair writes, advisors answer

The Discussion-Write row gains a second producer, `DiscussionSeats`, beside the unchanged single-agent `DiscussionWrite`: a chair that writes the discussion as the single writer does, plus one advisor seat per entry of `loom.yaml`'s new `discussion_advisors` list, each studying the task and answering the chair over the session message channel.
`loom.yaml`'s new `discussion_producer` key selects which of the two the row runs, defaulting to the single-agent producer, so a hub that sets nothing behaves as today.
The seat engine and the producer gain only additive seams, the config keys and the two new stencils land before the table builder that reads them, the fake seat runner learns to evaluate a gate, the registry entry and its wired table land under a coverage-guard bridge, and the selection then wires the entry into the row.
The decision record's carry-over entries were checked against the tree: the open-map entry for `discussion_advisors` is card 3's work, and the orch trust-rule entry is design text that no card touches.

## Card Index

1 — seat-table-optional-values — a seat table names the table values that may be empty, and exposes its composed prompts
2 — multillm-table-source-and-preparation — MultiLLMProducer takes a per-call table source and a fresh-spawn preparation
3 — loom-config-discussion-keys — loom.yaml gains discussion_producer and discussion_advisors, validated at load, the list carried whole through reconcile
4 — discussion-chair-and-advisor-stencils — the chair stencil includes the discussion stencil, the advisor stencil ships, and both register under their roles
5 — discussion-table-builder — loomengine builds the seat-table producer's table from the location, config and registry
6 — seat-runner-gate-once — the fake seat runner evaluates a table's gate once on a done chair, as the fake shuttle does
7 — discussion-seats-entry — the DiscussionSeats registry entry runs the told table on MultiLLMProducer behind the commit decorator, and loomcli wires the table
8 — discussion-write-row-selection — loomrecipe builds the Discussion-Write row with the engine loom.yaml selects, and loomcli wires the choice
9 — docs-chair-and-advisors — the overview and the recipe spec describe the two producers and the keys that select and shape them

## Shared Decisions

### Decision: two-producers-selected-by-config

- **Decision:** The parent's decision superseding D1 is implemented as a second registry engine, `DiscussionSeats`, built by its own `shedrecipe` entry over `Env.DiscussionTable`, while the `DiscussionWrite` entry, `DiscussionSpec`, `prompt.go` and their tests stay untouched.
  `loom.yaml`'s `discussion_producer` key takes `single` (the default) or `seats`; `loomcli` turns it into `Env.DiscussionSeats`, and `loomrecipe.New` rewrites the Discussion-Write row's engine in the parsed recipe before building, so the row name, its gates, its routing and the status file are the same whichever producer runs.
  D10's rename of the `discussion` role-map key is dropped: `discussion` keeps the single-agent stencil, and `discussion-chair` and `discussion-advisor` are added.
- **Rationale:** the row name is the durable resume identity and the recipe file is embedded, so the choice has to be applied where loom already applies `loom.yaml` to rows, the review budget in `loomrecipe.New`; a registry name of its own keeps the recipe registry's one-engine-per-build contract and the cross-consumer coverage guard honest, while the gate-quiescence guard, which D1's kept engine name would have left as it is, gains `DiscussionSeats` as a writer engine and a row over the substituted recipe in card 8.
- **Applies to:** cards 3, 7, 8, 9

### Decision: chair-stencil-includes-the-discussion-stencil

- **Decision:** The chair's opening stencil is a new `loom-template-discussion-chair` that includes `loom-template-discussion` whole and `seat-directive-chair`, then adds the advisors section D6 describes, opening with a sentence that replaces the included stencil's single-agent framing; the discussion stencil's agent-facing body is unchanged, and only its leading header comment, which every fill strips, gains the chair stencil's include as its second filler.
- **Rationale:** D6 put the chair seat directive into the discussion stencil, but the single-agent composer fills that stencil through `stencil.FillOptional` with no includes, which refuses an include it has no entry for, and the parent's decision keeps that composer as it is; an include carries D6's rationale (one source for the writer's fence and gates, no duplicated contract) without touching the single path.
  The seat engine fills an included block's markers from the same value map and refuses an include that includes, which the discussion stencil does not.
- **Applies to:** cards 4, 5

### Decision: absent-key-takes-template

- **Decision:** `discussion_advisors` normalises null, `[]`, `""` and `[""]` to no advisors at load, and an absent key takes the template's one advisor, because `configengine.Load` fills a missing key from the template before the unmarshal, as it does for every loom key; an absent `discussion_producer` takes `single` the same way.
- **Rationale:** PATTERN-config-strictness makes the fill the loader's contract; the record's "absent key loads as zero advisors" holds for an in-memory `Config`, which is what the table builder's tests use.
- **Applies to:** cards 3, 5

### Decision: resolve-at-load-and-in-the-builder

- **Decision:** `LoadConfig` parses and resolves every advisor entry, loading the registry through `modelspec.LoadRegistry` from its own base directory only when the list holds entries, and refuses an empty, malformed or unknown one naming the key, the index and the way forward, as D4 requires, whatever `discussion_producer` holds; `DiscussionTable` resolves each entry again for the seat's model, effort and version, as `discussion` resolves there today.
- **Rationale:** D4 makes an unresolvable entry a config refusal at load, and `LoadConfig` already reads `burler.yaml` from the same base directory to validate the fan keys; the builder takes a `Config` that need not come through `LoadConfig`, so it keeps `Resolve`'s error.
- **Applies to:** cards 3, 5

### Decision: no-parent-name-parameter

- **Decision:** `DiscussionTable` takes no parent name; the seat engine renders every seat's parent directive from `Geometry.ParentName`, which `wire` already tells it.
- **Rationale:** a parameter the builder cannot use would be dead.
- **Applies to:** cards 5, 7

### Decision: mode-rules-wrapped

- **Decision:** the advisor-aware mode text is `discussionModeRules`, which returns `modeRules`' text and appends the advisor sentence in autonomous mode with advisors; `modeRules` and `prompt.go` stay as they are.
- **Rationale:** the single-agent composer keeps `modeRules`, so the two mode texts stay in one place and the seat-table producer adds only its one sentence.
- **Applies to:** card 5

### Decision: fake-seat-runner-evaluates-the-gate-once

- **Decision:** `shedfake.SeatRunner` evaluates a table's gate once on a done chair whose answered result carries no `GateOutcome`, as `shedfake.Shuttle` does for a gated run, and lands in its own card before the row selection.
- **Rationale:** card 8's loomrecipe test proves the row's gate rides the chair by letting the real discussion gate judge the files the fake chair wrote; a `RunFn` that evaluates the gate itself returns its outcome, so the gate is never evaluated twice.
- **Applies to:** cards 6, 8

### Decision: coverage-guard-bridge

- **Decision:** card 7 adds `DiscussionSeats` to the coverage guard's allowlist, since no recipe row names it until card 8's substitution; card 8 adds it to `loomrecipe.RecipeEngines` and to the engine-set test's want set, and removes the allowlist entry, which the guard's drift check forces.
  Card 7 also fills the new func-typed `Env.DiscussionTable` in `loomcli`'s `wire`, since `TestWire_EverySeamFilled` refuses a nil seam `wire` does not list; card 8 adds only the `DiscussionSeats` choice.
- **Rationale:** each card builds and tests green on its own, and the guard's own drift rule makes the bridge impossible to leave behind.
- **Applies to:** cards 7, 8

### Decision: test-budget-trajectory

- **Decision:** `cmd/lyx/testdata/test-budget.yaml` rises by two on `internal/loomengine` in card 5, by two on `internal/shedrecipe` and by one on `internal/loomcli` in card 7, and by one on `internal/loomrecipe` in card 8; every other card adds rows or subtests to existing tests.
- **Rationale:** each of those packages is at its budget today, and the raise lands in the commit that adds the tests.
- **Applies to:** cards 5, 7, 8

### Decision: pattern-overview-untouched

- **Decision:** no card edits `PATTERN.md`, since the task adds no cross-cutting invariant, so the parent's overview-cap decision needs no offset.
- **Rationale:** the new behaviour rides existing invariants (stencil ownership, told geometry, the parent and edit directives, config strictness, refusal way forward) and adds none.
- **Applies to:** all cards

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
