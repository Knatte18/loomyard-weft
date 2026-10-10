---
format: 5
approved: true
language: go
---

# Plan: the darn task recipe and the four landing fixes it depends on

Add `darn`, a second embedded recipe the loom module drives: one gated LLM writer that reads the board entry, edits, tests and commits, then Publish, PR-Gate and Finalize, with a verify gate that re-prompts the same session up to `darn.yaml`'s budget and halts the run for the parent when it is spent.
The first four cards land the shared fixes the record names (#502's per-run verify log, #501's parent run-records commit in Finalize, the teardown's board-claim settle and the halt note's parent fabric status), the next two give the verify gate its told command and merge base, and the rest add the recipe from the vocabulary up to the operator-facing docs.

## Card Index

1 — verify-log-per-run — one `verify-<n>.log` per verify run with a bounded history, and the failure record names the run's own log
2 — finalize-parent-records — Finalize commits the parent pair's run records before its parent-side merge through a told seam
3 — teardown-settles-board-claim — the pair-teardown composite marks a landed run's board entry done and clears an abandoned run's claim
4 — halt-note-parent-fabric-status — a dirty-tree halt note lists the parent pair's dirty paths, read through fabricengine
5 — fabric-merge-base — a go-git merge-base read in gitrepo and its fabric form
6 — verify-gate-told-command — the verify gate takes a told command and an optional merge base, and landing takes its failed-verify clause from the recipe
7 — recipe-vocabulary — `darn` joins the seed vocabulary, the `lyx shed` table and the segment set, with its reed color
8 — darn-config — `internal/darnengine` owns the hub-wide `darn.yaml` and its verify-command reader
9 — darn-stencil-and-spec — the darn writer's stencil and the spec that launches it
10 — darn-producer-and-entry — the `Darn` row's producer, its `DarnWrite` registry entry and the embedded darn recipe
11 — darn-builder-and-routing — `loomrecipe.NewDarn` stamps the verify budget and refuses one below 1, beside `DarnRouting`
12 — loom-arms-darn — loom's arm builds, routes, seeds, wires and rejects per the seed's recipe
13 — batten-admits-darn — batten's child seeding admits `darn` beside `loom`
14 — docs-name-the-recipe — the orch stencil, the board skill and help, and the overview name the recipe

## Shared Decisions

### Decision: darnengine-owns-darn-config-and-spec

- **Decision:** `darn.yaml`'s config type, loader, template and verify-command reader, and the darn writer's spec composer, live in a new leaf package `internal/darnengine`, told every path and importing no resolver.
- **Rationale:** the registration oracle in `internal/configreg` pairs one `ConfigTemplate` with one package, as landing and loom each have; a second template inside `internal/loomengine` would be invisible to it, and the spec composer belongs beside the config it reads.
- **Applies to:** cards 8, 9, 12

### Decision: one-verify-gate-two-forms

- **Decision:** `loomshed.NewVerifyGate` stays the one verify gate: a nil merge base keeps the round form loom's Webster-Burler row runs, and a told merge base selects the whole-diff form the `Darn` row runs; the gate's `verify` name in a recipe resolves to it for both recipes, and `shedrecipe.Env` carries the command and merge-base seams the wiring tells.
- **Rationale:** D6 asks for one told command source across loom and darn, and the two forms share every findings renderer and the Publish failure confirmation; a sibling constructor would duplicate them and need its own gate name.
- **Applies to:** cards 6, 11, 12

### Decision: recipe-read-once-at-arming

- **Decision:** loomcli reads the seed's recipe once in `armAt`, right after the run-id, onto the receiver; the routing, the shed builder, the wiring branch, the landing verify source and the reject rework row all read that field, and no producer or generic verb reads the seed.
- **Rationale:** D1 names loom's arm as the one place the recipe selects the graph, and a single read keeps every verb, the lightweight ones included, on the same recipe.
- **Applies to:** card 12

### Decision: no-sandbox-change

- **Decision:** no sandbox suite file changes for `darn`.
- **Rationale:** the sandbox coverage guard is keyed on registered CLI modules, `darn` registers none, and the core suite's config scenario already covers `lyx config <module>`, which is how `darn.yaml` is exercised.
- **Applies to:** card 8

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
