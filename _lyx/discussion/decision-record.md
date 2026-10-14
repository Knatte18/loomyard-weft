# Decision record — plan-gate-gaps

## Goal

Webster's checks agree with the code a plan changes, so a correct batch is never rejected and a cross-package break is caught at the card that causes it.
Three gaps close:

1. Record-batch's done-checks reject a batch whose later card renames or deletes what an earlier card of the same batch targets (tender batch 07).
2. The batch gate names a package directory a later card of the same batch moved away, so `lyx gate test` refuses it (tender batch 07).
3. A card's gate runs only the card's own packages, so a repo-scanning test such as `TestGitrepoBoundary_PinnedRunCallSites` or `TestTierPurity_UntaggedTestsSpawnNothing` fails only at a later gate (issue #594, tender batch 01).

## Scope

In:

- A batch-boundary rule every grouping batchifier obeys: a card that renames or deletes what an earlier card of its candidate batch targets starts a new batch.
  This is the board's "the Batchifier puts a rename in its own later batch" route, and it answers scope items 1 and 2.
- A guard step at the end of every card's gate that runs every `//lyx:guard`-marked test of the root module by name.
- The guard-test scan that `internal/impactset` holds, made callable by webster so both share one marker scan.
- Docs in the same commit: `internal/batcher/doc.go`, `internal/websterengine/doc.go`, `internal/impactset/doc.go`, and `contracts/specs/loom-plan-spec.md` both where it describes batching and where it describes the per-card gate.

Out:

- Resolving glyphs at each card's own commit.
- Rewriting done-check, bind, drift or gate-scope inputs through the plan's renames.
- A verb to accept a batch by hand.
- A plan-time `validate-plan` check for symbols pinned by name.
- Guard tests in nested modules.
- Regrouping a run whose partition is already recorded in `state.json`.

## Decisions

### D1 — A rename or delete of an earlier card's target starts a new batch

Card B cannot share a batch with an earlier, distinct card A when one of B's Rename old sides or Delete targets covers one of A's targets (Create, Edit, Delete or Rename new side; `Uses` is excluded, since it is read and never done-checked or gated).
"Covers" is:

- a symbol-level old side or Delete target (a glyph or `plan:` handle naming a member) covers A's target that names the same member glyph or handle body;
- a file-level old side or Delete target covers A's target that is that file, or a glyph whose unit is that file, or a package-level glyph or handle whose package directory is that file's directory;
- a directory-level old side or Delete target covers A's target whose file or package directory lies at or under that directory.

A ref `planparser` cannot map to a unit never covers and is never covered, so it forces no split.
The shape of each ref (member, file, directory) and its unit come from `planparser`'s ref helpers (`RefScope`, `RefFile` or a new helper there), never from local string tests (`PATTERN-ref-shape-registry`, `PATTERN-glyph-conversion-chokepoint`).
The rule is one `internal/batcher` helper every grouping batchifier calls: the cost batchifier treats a segment holding such a pair as infeasible, a one-card segment stays always feasible, and the identity batchifier is unaffected since its batches hold one card.
It applies wherever the partition is derived, a first init and a rebaseline alike (`PATTERN-batcher-registry`).

Rationale: with B in a later batch, record-batch checks A's batch against a tree where A's targets still stand, and A's batch gate names directories that still exist, so gaps 1 and 2 close with one rule and no change to `DoneChecks`, `BindHandles`, `DetectDrift`, the scope guard or `gateScopeOf`.
B's own batch is checked by the existing rename and delete done-checks, and its gate already takes a Rename group's new side only.

Bound: the rule only adds batch boundaries.
It never merges batches, reorders cards or moves a card across a dependency, so card order and every forward dependency stay as the batchifier left them.
Its cost is one more fork and batch gate per such boundary; a file-level old side in a package an earlier card touched splits even when the renamed file is not the one that card edited, which over-splits rather than under-splits.

### D2 — Every card gate ends with the root module's guard tests

`cardGateCommand` appends one step after the card's own `lyx gate test` steps and before the two lints:
`lyx gate test --tags integration <guard packages> -- -run '^(<guard test names>)$'`.
The guard list comes from the same static `//lyx:guard` marker scan `internal/impactset` runs for the Webster-Burler-Fix round gate.
Webster finds the root module's package directories by walking the worktree for directories holding `_test.go` files, skipping any directory under a nested `go.mod`, so the render spawns no process.
begin-batch and recover-batch render it, since both render card gates through `cardGateCommand`.

A guard package directory that a card of the batch renames files out of or deletes, by D1's "covers" relation, drops out of the step for every card of that batch, so the step never names a directory the batch removes.

Rationale: the repo-scanning tests that broke in #594 and tender batch 01 are already marked `//lyx:guard`, and running every guard test of this repo took about 7 seconds wall on a warm build cache, so catching the break at the card that causes it costs seconds per card.
A runtime catch is generic across hubs where a plan-time check would have to know what each scanning test pins.

Bound: the step adds tests to the gate and skips none.
It runs every marked test, and compiles every test file of each guard package, so a compile-time pin there (such as `cmd/lyx/seamsignature_test.go`) breaks the step too.
An unmarked runtime pin, and a guard test an earlier card of the same batch adds, are caught only by the batch gate or the plan-level verify, as today.
A worktree with no `go.mod` at its root, or a root module with no marked test, renders no guard step.
A `-run` pattern joins names across packages, so a same-named unmarked test in a guard package runs too; that only adds a test.
A step longer than the command-line limit impactset already applies renders as D3's line instead.

### D3 — A failed guard scan renders a line, not a step

When the guard scan fails (a `GuardScanError` for a misplaced marker or an unparsable test file, a read error in the walk, or an over-long step), the card gate renders no guard step, and the fork prompt carries a line naming the error and saying the plan-level verify still runs the guard tests.
begin-batch does not refuse.
Error policy is per caller: impactset's round gate keeps returning a scan error as it does today.

Rationale: the fault sits in a test file the fork may or may not own, and refusing would stall the batch on a fault the plan-level verify already reaches; the line tells the fork what it is missing.

Bound: a scan error drops only the guard step for that batch's cards; the card's own gate steps, the lints and the plan-level verify run unchanged.

### D4 — A fork fixes a guard failure outside its card's targets

The fork prompt states that a guard step failure may require editing a file the card does not list, such as the pinned set in a `cmd/lyx` test, and that the fork fixes it under the card's commit.

Rationale: the glyph scope guard reports such a file as a deviation and never blocks, so the fix needs no new permission, only the instruction.

Bound: the instruction covers only a file the guard step's failure names; every other deviation rule stays as it is.

## Constraints

- `PATTERN-batcher-registry`: webster's execution unit stays the batch the active `batcher.yaml` profile derives through `internal/batcher`; D1 extends that derivation rather than adding a second grouping site.
- `PATTERN-ref-shape-registry`: D1 classifies refs only through `planparser`.
- `PATTERN-gate-slot` and the gate rule: the guard step is a `lyx gate test` command, so it runs inside a hub gate slot and never as raw `go test`.
- `PATTERN-stencil-ownership`: the fork-prompt wording for D3 and D4 lives in the webster stencil read through `internal/stencilstore`, never in Go.
  A test pinning it pins the marker or the rule's presence, not the sentence.
- `PATTERN-sole-parsers`: D1 reads cards through `planparser`'s parsed `Card`, never by reparsing card text.
- `PATTERN-test-speed`: the new tests are untagged unit tests over parsed cards, rendered strings and fixture test files under `t.TempDir`; none spawns a process.
- The recorded partition in `state.json` is never regrouped mid-run.

## Auto-mode assumptions

- A1: One extra fork per split batch is an acceptable price for correct done-checks.
- A2: About 7 seconds per card gate is an acceptable price for per-card attribution of a guard break.
- A3: Guard tests outside the root module are not needed yet.

## Open risks

- A guard test that is slow, or one that fails on a pre-existing break in the parent, fails every card gate of the run; the parent is required green before landing, which bounds the second case.
- The guard step adds one gate-slot acquire per card, so with the hub's slot count a fork waits more often, and a slot-busy exit is more likely.
- A repo-scanning test without the `//lyx:guard` marker is still caught late; coverage is as good as the marking.
- The guard step is Go-only; a .NET hub's card gates get no guard coverage (see `dotnet-scoped-verify`).
- D1's file-level rule may split batches the cost batchifier would have kept together, adding a fork's cost.

## Acceptance criteria

- A tender-batch-07-shaped plan (cards creating members of a package, then a later card renaming that package's files into a new directory) partitions with the rename card in a later batch than the creating cards under the cost batchifier, and both batches pass record-batch's done-checks.
- A later card's symbol-level Rename or Delete of a member an earlier card creates splits the batch; a later card that only edits the same package does not.
- A later card's Delete of a file an earlier card edits splits the batch.
- A card whose own Rename old side covers its own earlier target does not split from itself.
- A ref `planparser` cannot map forces no split.
- A card whose Rename old side covers nothing earlier in its candidate batch joins that batch as before.
- The identity batchifier's partition is unchanged, and a run resumed on a `state.json` partition recorded before this change keeps its batches.
- The batch gate of the batch before a rename names only directories that exist through that batch.
- A card gate in a worktree whose root module holds marked tests ends with one guard step naming every guard package and every guard test name, before the two lints.
- A batch whose card renames or deletes a guard package's files renders a guard step without that directory.
- A worktree with no root `go.mod`, or with no marked test, renders a card gate with no guard step.
- A misplaced `//lyx:guard` marker renders no guard step and a prompt line naming the scan error, and begin-batch still succeeds.
- recover-batch's rendered card gates carry the same guard step as begin-batch's.
- The rendered fork prompt carries D4's out-of-card-fix instruction.
- The impactset round gate's guard tests are unchanged by sharing the scan.

## Notes for the plan writer

- Done-checks: `planglyph.DoneChecks` in `internal/planglyph/donecheck.go`, called from record-batch's pass in `internal/websterengine/recordbatch.go`.
- Gate rendering: `gateScopeOf`, `cardGateCommand` and `renderBatchGate` in `internal/websterengine/cardgate.go`; `planparser.CardTargetDirs` already takes a Rename group's new side only.
- Ref scope: `planparser.RefScope` and `planparser.RefFile` in `internal/planparser/targetdirs.go`.
- Guard scan: `findGuardTests` in `internal/impactset/guards.go` takes its directories from `listPackages` (`go list`) inside `Derive`; webster passes its own walked directory list instead.
- Cost batchifier: `internal/batcher/cost.go`, the fewest contiguous segments that fit, with an early `break` once a segment stops fitting.
- The webster fork stencil that renders `card_gates` is where D3's and D4's wording goes.
