# Decision record — twb-d0-gc: Plan and Webster checks out of step with the code a plan changes

## Goal

Stop Webster from rejecting correct work because a card renames, moves or deletes, inside one batch, what an earlier card of that batch targets, and run a hub's repo-wide test packages in every batch gate, so the three scope items of the board entry are met.

## Scope

In:

- The cost batchifier (`internal/batcher`) never puts a card in the same batch as an earlier card whose targets it renames, moves or deletes.
- The conflict predicate between two cards, in `internal/planparser`.
- Gate scope drops a whole-file or package `Move` source, as it already drops `Delete` targets and a `Rename` pair's `Old` side.
- A new `webster.yaml` key, `batch_gate_always`, whose packages every batch gate runs in its own untagged `lyx gate test` step.
- Docs in the same commit: `internal/batcher/doc.go` (the conflict rule), `internal/websterengine/doc.go` (the extra batch-gate step), the `webster.yaml` template comment, `contracts/specs/loom-plan-spec.md`'s batching line gaining "moves", and `contracts/specs/refusal-spec.md`'s "config file content invalid" row.

Out:

- Resolving a card's glyphs at its own commit, and following renames in `DoneChecks` or `BindHandles`.
- A verb that accepts a batch by hand; D1 removes the known cause of the false rejection, and a false rejection from another cause still needs a fresh restart.
- Plan-time detection of tests that pin a symbol by name.
- `plan-drift-refusal-route`'s `plan_drifted` label and its `delete-before-reference` routing.
- The card gate's lint steps, the plan-level verify gate, and the identity batchifier, which already runs one card per batch.

## Decisions

### D1 — The cost batchifier keeps a card that removes an earlier card's target out of that card's batch

A card *removes* an earlier card's target when one of its removing refs overlaps a target of the earlier card.
Its removing refs are its `Rename` pairs' `Old` sides, its `Move` targets and its `Delete` targets.
Two refs overlap when they are the same ref, or when one's file or package scope contains the other's.
The earlier card's targets are its target groups' refs, a `Rename` group's both sides included; `Uses:` is excluded, since nothing checks it against the batch's final tree.
A cost-batchifier segment is admissible only when no card in it removes a target of an earlier card in the same segment.
A one-card segment is always admissible, a card removing its own targets included, so the batchifier's promise that no input leaves the split infeasible holds.
A plan without `Rename`, `Move` or `Delete` cards batches exactly as today.

Rationale: `postBatchChecks` (`internal/websterengine/recordbatch.go`) runs `DoneChecks` against the worktree after the batch's last card, and `BindHandles` against the batch's net delta, so an earlier card's `plan:` handle into a package a later card renames neither resolves nor binds.
`renderBatchGate` unions the batch's cards' directories at `begin-batch`, so it names a directory a later card of the batch renames away.
In the tender run, card 12 (`12-tender-rename.md`) carried a `**Rename:**` group of file pairs, `internal/pausecli/cli.go` -> `internal/tendercli/cli.go` and the rest, while cards 8–10 targeted `plan:` handles in `internal/pausecli` and `internal/pauseengine`; the file-pair scopes overlap those package scopes, so D1 splits card 12 from cards 8–10.
Splitting there makes each batch's final tree the tree its cards were written against, fixing scope items 1 and 2 with the rule `loom-plan-spec.md` already states for renames and deletes; a move removes its source the same way, so the spec line gains it.

Bound: the rule only splits batches, never reorders cards, so card order and every edge `CheckBatchOrder` asserts are kept.
The removing card leaves the batch of each earlier card it conflicts with, and may still share a batch with earlier cards it does not conflict with and with the cards after it.
Overlap over-approximates: a delete of one file in a package splits from an earlier card editing another file of that package, costing a batch and never a wrong verdict.

### D2 — The conflict predicate lives in `internal/planparser`

The predicate is a planparser function over the plan and two cards.
It compares `RefScope` outputs only, never raw glyph strings.
A `plan:` handle claimed as a `Rename` pair's to-side takes the unit of that pair's old glyph, as the plan spec's handle unit rule states, not its draft unit half.

Rationale: `PATTERN-glyph-conversion-chokepoint` and `PATTERN-ref-shape-registry` keep every ref-shape and glyph-to-path decision in `planparser`.

### D3 — Gate scope drops a whole-file or package `Move` source

`CardTargetDirs` counts a `Move` ref naming a whole file or a package as removed, the same as a `Delete` target, so a directory every target of which a card moves away drops out of that card's gate and its batch's gate.
A `Move` of a member stays an ordinary target, since its source file survives.

Rationale: a `Move` card's target list holds its source and its destination is in `Intent`; without this a package `Move` renders its own vanished source directory into its gates, and `lyx gate test` refuses it.
A moved package's destination is not in the gate, which the plan-level verify gate covers.

### D4 — A recorded partition is not re-checked against D1

`ExecutionBatches` keeps mapping a recorded partition as it is, and no new refusal is added; D1 is pinned by the cost batchifier's own tests.
A rebaseline re-batches only the unbegun tail, with the active batchifier, so the tail follows D1.

Rationale: re-checking would wedge in-flight runs on a rule they were not formed under.

### D5 — `batch_gate_always` adds an untagged gate step to every batch gate

`webster.yaml` gains `batch_gate_always`, a list of root-module package arguments, for a Go hub; the template ships it as the empty list `[]`, so `--set` accepts a flow list for it, and a layer file without the key loads as empty.
`renderBatchGate` renders, after the batch's own `--tags integration` steps, one `lyx gate test <packages>` step over the listed packages, untagged.
It drops a listed package that the batch's integration step already covers, one whose directory holds no `.go` file at `begin-batch`, and one that a card of the batch removes, by D2's predicate.
A batch with no Go package of its own still renders the step when a listed package survives, so a stencil-only batch still runs the hub's scans.
`begin-batch` and `recover-batch` both render it, since both call `renderBatchGate`, and so does any later spawn path built on it, such as `weaver`'s.

`LoadConfig` refuses an entry that is not `./` followed by one or more slash-separated path segments, refusing `...`, a `..` or `.` segment after the prefix, whitespace, and a leading `-`.
The refusal is `refusal-spec.md`'s "config file content invalid" row, whose trigger is extended to name this key; its message names `webster.yaml`, the entry and the allowed shape, and gives that row's way forward, fixing the named file then re-running the verb.

Rationale: the pinned method set (`cmd/lyx/gitrepoboundary_test.go`) and the tier-purity scan (`TestTierPurity_UntaggedTestsSpawnNothing`) both live in `./cmd/lyx`, which holds the repo's cross-package boundary and PATTERN scans.
`lyx gate test ./cmd/lyx` ran in 8 s wall in this worktree, so running it once per batch is cheaper than any plan-time prediction of which tests a change pins, and catches what no prediction can, such as a new untagged `time.Sleep`.
Untagged, because the scans are untagged and `./cmd/lyx`'s integration tests run in the plan-level verify gate.
A config key keeps the package choice out of webster's code; a hub that keeps no such package sets nothing.

Bound: the step only adds a test run inside the fork's batch gate, under `PATTERN-gate-slot`'s `lyx gate test`; it skips nothing, and the entry shape check keeps a value from widening the command past named packages.

This hub's value is `[./cmd/lyx]`, on the board's repo-default layer, which this task's branch does not carry.
The PR description names the command, `lyx config webster --layer default --set 'batch_gate_always=[./cmd/lyx]'`, run from the prime after landing, and `lyx config webster` read back to confirm it.

## Constraints

- `PATTERN-batcher-registry`: the batch stays the one the active profile derives through `internal/batcher`; D1 changes the cost batchifier, never webster's call sites.
- `PATTERN-gate-slot`: every added step is a `lyx gate test` command, never raw `go test`.
- `PATTERN-config-strictness`, `PATTERN-config-layers`: `batch_gate_always` loads through webster's existing `LoadConfig`.
- `PATTERN-refusal-way-forward`: D5's refusal lands with its extended row and a test that reaches it.
- `PATTERN-test-economy`, `PATTERN-test-speed`: new behavior goes in as rows of existing table tests where one covers the function, at the lowest tier.

## Auto-mode assumptions

- `./cmd/lyx` is this hub's package for `batch_gate_always`; its cost was measured once.

## Open risks

- A rename card mid-plan now costs one more batch than before, and each extra fork starts deeper by `batch_growth`.
- A boundary test outside `batch_gate_always`'s packages still surfaces only at the plan-level verify gate.
- D5's hub value is set only when the orch or the operator runs its command after landing; a landing through the alignment agent's PR-Gate approve has no orch turn, so the operator runs it then.
- A run partitioned before this change keeps its batches (D4), so it can still hit the false rejection; its way forward is a fresh restart of Webster.

## Acceptance criteria

- The cost batchifier never forms a batch holding a card and a later card that renames (`Rename` `Old` side), moves or deletes one of its targets, whether the refs are equal or one's file or package scope contains the other's.
- The tender case: earlier cards with `plan:` handles in `internal/pausecli` and a later card renaming that package's files pair by pair are batched apart, and the earlier cards' batch gate names only packages present at that batch's end.
- A plan with no `Rename`, `Move` or `Delete` card batches as before.
- A card that names the removed target only in `Uses:` does not force a split.
- A card whose `Rename` or `Delete` overlaps one of its own targets still forms a batch, alone or with others.
- A `plan:` handle claimed as a `Rename` to-side is compared under its old glyph's unit.
- A rebaseline's tail batches follow D1, both when the removing card is the tail's first card and when both conflicting cards are in the tail.
- A recorded partition that predates the rule still maps and runs.
- A package `Move` card's card gate and batch gate do not name its source directory; a member `Move` keeps its directory.
- `renderBatchGate` renders the untagged `batch_gate_always` step after the integration steps; drops a listed package already in the integration step, one with no `.go` file, and one a card of the batch removes; renders the step for a batch with no Go package of its own; renders no step for an empty list.
- `recover-batch`'s prompt carries the same step.
- `LoadConfig` refuses a `batch_gate_always` entry without the `./` prefix, with `...`, with a `..` or `.` segment, with whitespace, or with a leading `-`, with the extended refusal row's message.
- A `webster.yaml` layer without the key loads `batch_gate_always` as empty, and the template ships it as `[]`.
- The PR description carries D5's command.

## Notes for the plan writer

- `internal/batcher/cost.go`'s dynamic program grows each segment backward, from `end-1` down, so an inadmissible segment makes every longer one ending at the same card inadmissible, and the loop can `break` as it does on the budget.
- `gateScopeOf` and `renderBatchGate` are in `internal/websterengine/cardgate.go`; `CardTargetDirs` is in `internal/planparser/targetdirs.go`.
