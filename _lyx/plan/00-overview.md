---
format: 5
approved: false
language: go
---

# Plan: MultiLLMProducer, one producer runs a chair and its advisors as seats

Build a Shed producer that runs N agents in one step, each in its own strand: one chair whose turn end, outputs and gate decide the step as a single agent's do today, and advisors whose outputs are the chair's inputs and who talk to the chair over Claude Code's session message.
The seat-running logic lives in a new `internal/seatengine` below `internal/shedadapters`, which gains the thin `MultiLLMProducer`; the recipe registry gains a `MultiLLM` row over a seat table, `internal/stencil` gains a one-level include form for the two shipped seat blocks, and shuttle gains the quiet-hold knob and the per-run send lock the seats need.
No loom row uses the producer yet; it is proven by its own tests, a fixture recipe and one `llm`-tier smoke run by hand.

## Card Index

1 — shuttle-quiet-hold-send-lock — Spec.QuietHold, the per-run send lock and Run.StrandName in shuttle
2 — stencil-include-form — FillWith and IncludeNames beside Fill, one level deep, markers guaranteed inside includes
3 — seat-directive-stencils — the two shipped seat blocks under contracts/stencils/seat, registered and marker-pinned
4 — seatengine-table — the new package's table, validation, told geometry and handle-shaped shuttle seam
5 — seatengine-prompts — per-seat value maps, directives, include-filled prompts and shuttle specs
6 — seatengine-run — fresh start in order, join, the chair-side notice sender and the stop rule
7 — seatengine-probe-resume — probe before archive, resume a live chair, tell it of advisors the probe lost
8 — shedfake-seat-runner — the shared fake seat runner the adapter and recipe tests drive
9 — shedadapters-multillm — the fifth adapter mapping the chair's result onto the Shed contract
10 — shedrecipe-env-seats — Env.Seats and Env.Models, filled by loom's wiring and the test fixture
11 — shedrecipe-multillm-entry — the MultiLLM registry row building a seat table from recipe config
12 — multillm-fixture-recipe — a fixture recipe built through shedbuild and run against fakes
13 — seatengine-llm-smoke — the hand-run llm-tier chair-and-advisor smoke
14 — docs-overview — the module table and execution stack name seatengine and the fifth adapter

## Shared Decisions

### Decision: probe-stops-and-adapter-archives

- **Decision:** `seatengine.Engine.Probe` stops every live advisor itself when the chair is not live and answers an empty `LiveTable`; the adapter then archives every seat's stale outputs with `archiveStaleOutputs` and calls `Run`.
  The engine never archives, so the shared helper stays in `shedadapters` and the probe-before-archive ordering is the adapter's, as `SingleLLMProducer` and `BurlerProducer` keep it.
- **Rationale:** D8 wants no advisor strand left without a handle to stop it, and the only package holding the archive helper is the adapter layer.
- **Applies to:** cards 7, 9

### Decision: handles-not-results-at-the-seam

- **Decision:** the `shedadapters.SeatRunner` seam and the `seatengine.Shuttle` seam are handle-shaped (`Probe`, `Resume`, `Run` over a `Table`; `StartGated`, `ProbeGated` over a `Spec`), copying burler's shape rather than the result-shaped `shedadapters.Shuttle`, because the engine must hold several live runs at once and send into the chair's while it waits.
- **Rationale:** D12.
- **Applies to:** cards 4, 8, 9, 10

### Decision: reserved-markers-declared-once

- **Decision:** the reserved marker list is one package-level declaration in `seatengine`, exposed by `ReservedMarkers`; the table's collision check, the value map and the recipe entry all read it.
- **Rationale:** D3's list is a contract three sites read, so it is written once.
- **Applies to:** cards 4, 5, 11

### Decision: env-fields-and-wiring-land-together

- **Decision:** `Env.Seats` and `Env.Models` are added in the same card that fills them in `loomCLI.wire`, because `TestWire_EverySeamFilled` reads every `Env` seam and fails on an unfilled one.
- **Rationale:** card gates run the whole untagged suite, so the two edits cannot be split across cards.
- **Applies to:** card 10

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
