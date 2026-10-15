## Goal

A plan whose later card renames, moves or deletes what an earlier card targets runs through Webster without a false done-check rejection and without a gate step that names a package the batch removed.
A change that breaks a repo-wide scan or a by-name pin in a hub-named package such as `cmd/lyx` fails at the batch that causes it, not first at the plan-level verify gate.
The board entry's scope items 1 and 2 are met through its third way out, the rename in its own later batch (D1, with D2 for a card's own gate); item 3 through D3.

## Scope

In:

- A batch barrier in `internal/batcher`: no batch holds a card that removes what an earlier card of the same batch targets, under every batchifier kind.
- Card and batch gate scope in `internal/websterengine/cardgate.go` that drops a directory no Go file survives the given cards in.
- A hub-configurable list of Go packages every batch gate also tests, in `webster.yaml`.
- The matching `doc.go`, plan-spec and config-template text.

Out:

- Resolving glyphs at each card's own commit, or following renames inside `DoneChecks` or `BindHandles`.
- A verb to accept a rejected batch by hand.
- Regrouping a partition already recorded for an in-flight run.
- Plan-time detection of by-name pins (`validate-plan`, Plan-Review).
- The `plan_drifted` label and the `delete-before-reference` routing, which `plan-drift-refusal-route` owns; that check is an earlier delete with a later reference, the mirror of D1's earlier target with a later removal, and D1 changes nothing it reads.
- Gate commands for a non-Go hub, which `dotnet-scoped-verify` owns.
- Setting this hub's `batch_gate_packages` value: the operator or orch sets it after the landing deploys (D3).

## Decisions

### D1. The batcher enforces a removal barrier inside every batch

Decision: a contiguous segment of two or more cards is infeasible when a card in it removes what an earlier card of the segment targets.
A card's removed set is its `Delete` refs, its `Rename` pairs' `Old` sides and its `Move` refs; a `Custom` card never counts as a removal, an accepted gap of the escape hatch.
An earlier card's target set is its flat target list, both sides of each `Rename` pair included.
A removed ref hits an earlier target when the removed scope contains the target's scope (a member glyph by equality, a file by its members, a directory or package by everything under it), or when the earlier target is a package or directory scope that contains the removed scope.
Scopes come from `planparser.RefScope`; a ref it cannot map never hits.
The rule lives in one `internal/batcher` function that the cost batchifier's segment feasibility calls; identity satisfies it by construction.
When every adjacent pair is barred, the cost batchifier returns one-card batches, never an error.
Every call to a batchifier applies it, a re-partition after `rebaseline`, a reset or `run --fresh` included.

Rationale: the deployed hub plan spec (`_board/_lyx/specs/loom/loom-plan-spec.md`) states "A card that renames or deletes what an earlier card targets never shares that card's batch"; the spec source at HEAD, `contracts/specs/loom-plan-spec.md`, lacks the sentence, and no batchifier at HEAD enforces it.
With the barrier, every batch's done-checks and handle binding run against a tree where its own cards' targets are final, so `DoneChecks` stays a single working-tree resolve.
The second hit rule exists because a glyph on a package unit does not say which file its member lives in, so a file moved out of that package may carry it.
In the `tender` incident, cards 8–10 targeted `plan:internal/pausecli#…` and card 12 renamed `internal/pausecli/*.go` by file `Rename` pairs, a hit under the second rule, so card 12 lands in a later batch and batch 07's gate no longer names `./internal/pausecli`.
A false hit costs one extra batch; a missed hit costs a halted run.

### D2. Gate scope counts the Go files that survive the cards

Decision: `gateScopeOf` keeps a directory only when, after the given cards in order, it still holds a Go file: the `.go` files on disk when the gate is rendered, minus files a card removes (a `Delete` file ref, a file `Rename` pair's `Old` side, a `Move` file ref, every file under a package `Rename` pair's `Old` side), plus files a card creates (a `Create` file ref or Go-naming target, a `Rename` pair's `New` side).
A `Move` of a symbol removes no file.
The card gate and the batch gate both go through it, at begin-batch and at recover-batch alike; removing a file already gone or creating one already present changes nothing, so a recovery rendered after some cards committed gets the same scope.

Rationale: D1 fixes the incident's batch gate; D2 covers a removal card's own gate.
Today `gateScopeOf` reads `.go` presence as the tree stands at render time and a `Move` ref counts as a live target, so a whole-package `Move` or package `Rename` card's own gate names its vanished source, which `lyx gate test` refuses.
A `Move` destination lives in `Intent` prose, so no gate names it; the plan-level verify gate covers it.

### D3. `webster.yaml` names Go packages every batch gate also tests

Decision: a new `webster.yaml` key, `batch_gate_packages`, holds root-module Go package arguments for `lyx gate test`, empty in the template.
Each entry is `.` or starts with `./`, and holds no whitespace and no leading `-`; webster's config load refuses any other value with a way forward naming `lyx config webster --set` (`PATTERN-refusal-way-forward`).
`ConfigSpec` classifies it `CategoryGuard`, since emptying it drops a gate step, so the local layer may not hold it.
When it is not empty, the rendered batch gate ends with one untagged `lyx gate test <packages>` step, in begin-batch's and recover-batch's prompts alike, also for a batch with no Go package of its own; an entry the batch's own root-module scope already holds is left out of that step.
Card gates do not get it.
Each begin-batch and recover-batch reads it afresh, as `kickstart_pack` is read.
A failure in that step is the batch's to fix under the same self-fix and recovery rules as its own packages.
The key enters the board's repo default through the template and the hub reconcile; for this hub the operator or orch sets it to `./cmd/lyx` after the landing deploys, with `lyx config webster --set`, never inside the running task.

Rationale: the repo-wide scans (`TestTierPurity_UntaggedTestsSpawnNothing`) and the by-name boundary pins (`cmd/lyx/gitrepoboundary_test.go`) sit in `cmd/lyx`, and no card's own package list reaches them.
Which package holds such scans is a property of the repo, not of lyx, so it is hub config, not a constant.
The step is untagged because those scans are untagged tests, and `--tags integration` over `cmd/lyx` would add its integration tier to every batch.
Bound: the key only adds `lyx gate test` steps over validated package arguments; it skips and weakens nothing, and a value outside the grammar never reaches a prompt.

### D4. Docs land with the code

Decision: `contracts/specs/loom-plan-spec.md` gains the barrier sentence, naming `Move` beside rename and delete, and the webster template comment documents `batch_gate_packages`.

## Constraints

- `PATTERN-batcher-registry`: the barrier is part of how the batch is derived, inside `internal/batcher`, never a begin-batch regrouping; Weaver, which reuses the partition and the recovery prompt, inherits both unchanged.
- `PATTERN-config-layers` and `PATTERN-config-strictness`: the new key goes through `configengine` and the webster template; websterengine stays a degrading caller.
- `PATTERN-binary-change-reconcile`: the key reaches the board's repo default only through the hub reconcile.
- `PATTERN-test-speed` and `PATTERN-test-economy`: unit-tier tests, extending the existing batcher, cardgate and webster config tables where they fit.

## Auto-mode assumptions

- The barrier over the per-commit and rename-following alternatives (see the support log's ledger, Q1).
- `Move` counts as a removal for the barrier and the gate scope; `Custom` does not (Q2).
- `batch_gate_packages` over plan-time pin detection (Q3).
- No begin-batch re-assertion of the barrier, so a partition an older binary recorded keeps running (Q4).
- `CategoryGuard` for the key, and the hub value set after deploy (Q6).

## Open risks

- The prod-deployed spec carries the barrier sentence that HEAD's source lacks, so a change on the parent may already touch the batcher; Merge-In reconciles it.
- No measured untagged `cmd/lyx` run time exists; each batch of this hub pays one such run, recorded in the fork's gate output, and the operator can empty the key.
- A `cmd/lyx` already red at the run's base fails every batch's gate; the plan-level verify gate stops such a run today too, only later.

## Acceptance criteria

- A plan whose card creates `plan:<pkg>#X` or targets `<pkg>#X` and whose later card renames or moves `<pkg>`'s files never lands both cards in one batch, under the cost batchifier at any budget.
- A card that creates `X` and a later card that deletes `X` never share a batch.
- A later card that deletes a member of a file an earlier card edits a different member of shares a batch with it when the cost model allows.
- An unmappable ref never forces a split, and a `Custom` card never counts as a removal.
- A plan whose every adjacent pair is barred gets one-card batches from the cost batchifier, with no error; the identity batcher's output is unchanged.
- A card that moves or file-renames every Go file out of a directory, or renames its package, renders neither a card gate nor a batch gate step naming that directory; a directory with a surviving Go file stays.
- recover-batch rendered after the removal card committed names neither the removed directory nor a missing one.
- With `batch_gate_packages` set, begin-batch's and recover-batch's batch gates end with the untagged step, including for a batch with no Go package, and leave out an entry the batch's own scope holds; with it empty, the gate renders as before.
- A `batch_gate_packages` entry with whitespace, a leading `-` or no `./` prefix is refused at config load with the way forward.

## Notes for the plan writer

- The cost batchifier's segment feasibility is in `internal/batcher/cost.go`.
