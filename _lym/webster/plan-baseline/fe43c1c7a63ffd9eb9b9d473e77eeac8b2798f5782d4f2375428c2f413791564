---
format: 5
approved: true
language: go
---

# Plan: loom names a contract, not a recipe

`loom` stops naming one recipe and names a contract: a recipe declaring `contract: loom` is checked by Shed at build, `lym loom` serves loom runs only (pawl excepted until it joins), and `tapestry` and darn share one validator, one gate, one builder, one Describe and one writer wrapper, driven by the recipe's rows and keys.
Darn's decision record is written by a Go row from the board entry, the config splits into a shared `loom.yaml` and a new `tapestry.yaml`, the shed driver keys live in `shed.yaml` alone, and the Tapestry-Preflight row and the `beam` alias go.

## Card Index

1 — discussion-fence-rule — the discussion parser takes CommonMark's top-level fence rule, and the section check honours it
2 — drop-beam-alias — a seed recording beam reads as the unknown recipe beam
3 — loom-contract-check — Shed checks a recipe declaring contract: loom at build; tapestry declares it
4 — retired-key-way-forward — a retired config key carries a way-forward text its local refusal appends
5 — shed-keys-in-shed-only — the arbiter, tackler and driver-kind keys leave loom.yaml and darn.yaml and are refused there
6 — tapestry-config-module — the plan and implementer keys move to a new tapestry.yaml module
7 — recipe-key-accessors — loomrecipe answers a recipe's contract, keys, record engine, rows and retired rows by name
8 — one-loom-builder — one recipe-keyed New, Routing and RetiredRows replace the tapestry and darn pairs
9 — drop-tapestry-preflight — Tapestry-Preflight and its seed check go; Preflight loads batcher.yaml for an Implementer recipe
10 — drift-verify-before-review — tapestry runs Drift-Verify after Parent-Align and before the Code-Review perch
11 — record-write-row — a Go Record-Write row writes darn's record and support log from the board entry
12 — darn-review-answer-key — darn's Code-Review perch measures against the decision record, with no snapshot
13 — one-discussion-validator — one validator and gate for every loom record, and Darn-Write's post-session guard
14 — shared-writer-wrapper — Discussion-Write and Darn-Write share one run-and-commit wrapper
15 — darn-write-scope-guards — Darn-Write refuses a missing record, halts on its writer's stop marker, and stops writing the record
16 — decision-add-by-rows — decision add keys on the recipe's rows and names reachable ways forward on a Record-Write record
17 — one-describe — one Describe table and stencil serve every loom recipe
18 — recipe-keyed-wiring — loomcli's remaining recipe-name branches read the recipe's rows and keys
19 — loom-verbs-loom-only — lym loom refuses a non-loom run, and a plan verb refuses a recipe without a plan
20 — peers-per-recipe — lym loom peers reads each peer against its own recipe

## Shared Decisions

### Decision: recipe-reads-through-loomrecipe

- **Decision:** Every Go branch that used to compare a recipe name reads the recipe's rows or keys through card 7's loomrecipe accessors, an empty recipe reading as tapestry; pawl's branches stay under D2's exception, named once in `loomrecipe.ServedByLoom`.
- **Rationale:** D8 drives each difference from the recipe file, so a third loom recipe is a recipe file, not new branches.
- **Applies to:** cards 7, 8, 9, 13, 16, 18, 19, 20

### Decision: tapestry-module-in-loomengine

- **Decision:** The `tapestry` config module lives in `internal/loomengine` beside loom's, with its own template file, Spec, loader and registry entry, and loom's loader fills its plan fields from it, so the readers of those values keep their signatures.
- **Rationale:** The module's load must read `loom.yaml`'s run-wide review keys (D12), and loom's plan table and review resolution read the moved values; a separate package would need an import cycle or a signature change at every reader.
- **Applies to:** card 6

## PATTERN changes

- `PATTERN-loom-contract` — adds: Adding a loom recipe: it declares `contract: loom`, checked by Shed's recipe loader; `lym loom` verbs serve loom runs only, bar pawl until it joins; a verb for some loom recipes refuses on others by the recipe's rows, never its name.
- `PATTERN-shed-stop-chain` — changes: A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; a stop the driver's arbiter seam marks as judged first, by row and reason, skips the tackler; arbiter keys in shed.yaml, batten's overridable in batten.yaml, a loom recipe's own via its profile block.
