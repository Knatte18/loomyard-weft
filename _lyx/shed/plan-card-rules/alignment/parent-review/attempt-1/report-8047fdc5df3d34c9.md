---
verdict: APPROVED
---

## Scope check against the board entry

The board entry `plan-card-rules` has no Done-when section; its operator scope line of 2026-10-22 names four points, and the decision record covers each one, with nothing added and nothing left out.

1. Plan-gate headroom check: D1 projects the overview from the cards' verbatim lines and runs `pattern.Headroom` against the merge base, and D2 stops a plan from getting around that check by describing a declared entry in prose.
   The entry's "or the Discussion stage" was only a suggestion; the operator's scope line names the plan gate, so leaving a Discussion-stage check out is no deviation.
2. The `Extends` label, its card-DAG rule and its plan-review check, with the `tools/plandag` measurement first: these are D3, D4, D6 and D5.
   D5 puts the measurement card first and doesn't make the build wait for a rerun.
   That matches the operator line, which puts the label itself in scope, and the orch's 2026-10-18 estimate in the entry already reports the drop.
   The `fabriccli#addWeftVerbs` side finding stays out, as the entry says it should.
3. An in-run owner for a plan amendment: D7 (the Plan-Review fix loop over `--amends-plan`) and D8 (the arbiter and orch routes).
4. Enclose's first use only: D9 traces a Merge-In or Publish verify failure to plan cards, and the Publish failure note, which the fixer reads, names them.
   Every later Enclose use the entry lists stays out.
   Quarry v0.4.0 is already pinned in `go.mod`, so the v0.3.0 bump the entry mentions isn't needed.

## Other board entries

- `parallel-batch-parts` and `weaver`: no duplication.
  `Extends` changes only which pairs count as dependencies, and the pair rule sits where their batchifier can call it; batches, worktrees and merge-back stay out.
- `shed-arbiter-all`: no contradiction.
  D8 keeps the arbiter stencil recipe-blind and puts the amendment target in the stop envelope, so moving the arbiter into shed still works.
- `run-verbs-shared`: no contradiction.
  `--amends-plan` is a flag on the existing `lyx loom decision add`, refused on a darn run, which has no plan; the namespace question stays with that entry.
- `speed-loom-profile`: no contradiction; the record lists it as an open risk, since a profile without the Plan-Review perch would leave an amendment with no fix loop.
- `kickstart-pack-remove`: honoured, since D9 builds no pack of failing members.

## Overlaps with runs in flight and parent commits

- `batten-cancel-verb` (row 5, Discussion-Burler-Review; this run's own row is 3).
  Overlap: both runs touch the orch role stencil.
  `batten-cancel-verb` changes its run-handling line to name `lyx batten cancel`, and this run adds the `--amends-plan` route for a design call that amends the plan.
  Both runs also touch the stop chain's code at different places: the peer adds a precondition row to `internal/sheddriver`'s rule table, and this run adds the amendment target to the loom arbiter's stop envelope.
  These edits sit at different places and change different behaviour, so this is a text-only overlap and doesn't conflict.
  The peer's `PATTERN-pair-teardown` change and this run's PATTERN constraints don't share an entry.
  Landing order: `batten-cancel-verb` is further along (row 5 against 3), so it lands first, and this run takes its changes at Merge-In.
- Parent commits: `lyx loom peers` lists none.
