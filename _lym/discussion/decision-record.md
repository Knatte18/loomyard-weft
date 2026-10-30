# Decision record — pawl-starts-at-preflight

## Goal

A new beam's pawl run starts at the row its seed names.
The shared inner-run seed sets `CurrentProducer` to `Preflight`, and the pawl recipe gets a `Preflight` row as its entry, so the seed and the recipe agree.

## Scope

- `contracts/recipes/pawl-recipe.yaml`: a `Preflight` row, engine `Preflight`, as the recipe's entry, with `on_done: Beam-Up`; the header comment names it.
- `internal/boardengine/runlock.go`: `Preflight` joins the hold-free rows, since it stands before Parent-Align.
- Tests that pin the pawl recipe's rows: `internal/loomrecipe/pawlrecipe_test.go` and `internal/loomcli/wiring_beam_test.go`.
- Package docs that spell the pawl recipe's rows or the hold-free rows: `internal/loomrecipe/doc.go`, `internal/loomcli/doc.go`, `internal/boardengine/doc.go`, and the other docs that list where a pawl run stands with no hold.

Out of scope: the seed, which stays shared and unchanged, and the wording of the "producer list has changed" refusal.

## Decisions

### D1 — Pawl's entry is the general Preflight row

The pawl recipe gains a first row named `Preflight`, engine `Preflight`, the general producer in `internal/preflightshed`, and `entry: Preflight`; its `on_done` is `Beam-Up`, and it carries no `on_stuck`, as in darn and loom.
Rationale: the operator's design; loom and darn start the same way, and `loomshed.Seed` stays the one seed for every inner recipe.

### D2 — A pawl run at Preflight holds nothing of its beam

`Preflight` joins `holdFreeProducers` in `internal/boardengine/runlock.go`, so a beam project whose run stands at `Preflight` takes members and scope edits, as it does at `Beam-Up`.
Bound: the lift applies only to a beam project, whose run is a pawl run; a loom or darn task at `Preflight` still holds its entry, because `RunHolds` lifts the hold for beam projects alone.
Rationale: the hold boundary is Parent-Align, and `TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign` pins every row before it as hold-free.

### D3 — In-flight runs need no migration

A pawl run already past its seed stands at a row the recipe keeps, so its status file stays valid; a run halted at the seed's `Preflight` now resumes at the new row.
Rationale: no row is renamed or removed.

## Constraints

- `PATTERN-shed-recipe-registry`: the row reuses the existing `Preflight` registry entry; no new constructor.
- `PATTERN-test-speed`: test changes stay in the existing tier-0 tables.

## Auto-mode assumptions

- The general Preflight checks pass in a freshly made beam pair, as they do in a darn pair.

## Open risks

- A beam pair left dirty by Worktree-Create would now stop at Preflight instead of Beam-Up; the stop names the check that failed.

## Acceptance criteria

- D1: the pawl recipe builds with `Preflight` first, of the general preflight producer's type, handing to `Beam-Up`; the wiring test's routing rows start with `Preflight`.
- D2: `RunHolds` is false for a beam project at `Preflight`, and the hold-boundary test passes over the new row list.
- D3: no retired-row mapping is added, and the existing pawl rows keep their names and edges.
