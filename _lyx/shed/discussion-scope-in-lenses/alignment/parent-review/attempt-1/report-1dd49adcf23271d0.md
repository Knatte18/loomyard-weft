---
verdict: APPROVED
---

The decision record meets every goal point of the board entry `discussion-scope-in-lenses`, takes no other entry's work, and conflicts with no run in flight.

## Goal points against the record

- Part 1, alignment agent dropped at the discussion point: D1 and D2.
  The entry's "keep it at `pr-gate`" names a row that `fc75aef79` removed: the loom recipe has no PR-Gate row and `loom-template-alignment.md` runs only at `parent-review`.
  D1 removes the whole agent, and the record's first Open risks line puts the leftover question to the parent.
  The board's own text already treats landing-time alignment as the Merge-In fixer's job: the entry's Part 3, and note `landing-escalate-arbiter`, which speaks of "the old alignment agent".
  So I raise no finding and do not escalate.
- Part 1, `board-fit` reads the in-flight runs and the parent's landed commits: D6.
- Part 1, a finding class that goes to the parent and not the fixer, routed by the Bouncer: D7 and D8, with the arbiter refusing it so that only the operator settles it.
- Part 1, the `parent-review` gate, `lyx loom review` and the parent-review name go: D2.
  D3 and D4, which retire the config keys and keep archived records readable, follow from the removal.
- Part 2, an outdated entry goes to the parent through Part 1's route: D6 and D9.
  Ordinary misalignment stays with the writer, as the entry requires.
- Part 2, Darn-Write checks its entry against the board and the parent's landed commits, then asks the parent and waits: D10.
- Part 3, the rename `Merge-In` to `Parent-Align` across the row, engine, producer, config keys, stencils and docs: D11.
  The entry says a run parked on `Merge-In` is restarted, while D11 carries such a run across with `shedengine.RenameProducers`.
  The entry gives the restart as a consequence of today's rule and does not rule out carrying a run across.
  The `Darn` → `Darn-Write` rename is the precedent, and the gentler path for in-flight runs is a design detail for Discussion-Review.
- Scope Out leaves fabric's merge-in, the Plan and Code perches' `decision` handling and the `twill-discussion` membership alone, as the entry asks.

## Board entries that touch the task

- `creel-agents`, `run-verbs-shared`, `regression-baseline-verify`, `landing-escalate-arbiter` and `loom-beam-darns` name the parent-review gate, `lyx loom review`, the alignment agent or `Merge-In`.
  The operator's entry orders those removed or renamed, and Scope Out leaves the rewrite of those notes to the orch.
  None of these entries is a running task.
- `lens-overlap-audit` recommends merging `goal-scope` and `board-fit`.
  It is an audit note with no decision, and the support log's rejected alternatives leave the fan trim to it.
- No board note titled `Decision for discussion-scope-in-lenses:` exists.

## Overlaps with runs in flight and parent commits

- `umbrella-daemon` (row 3, own_row 3: a tie, so there is no landing order).
  Overlap: both change `internal/arbiterstop`.
  `umbrella-daemon`'s D8 moves the arbiter into shed and adds an eligibility rule for batten, leaving loom's and darn's rules as they are.
  This run's D8 adds a loom refusal for the new `decision` cause.
  The two changes are different behaviours and they compose.
  Both also edit the orch role stencil and the landing texts that name `Merge-In`, and both retire `loom.yaml` keys through `hubreconcile`, but different keys.
  Each record names the other and says that whichever lands second takes the other's change, so the designs do not conflict.
  Neither changes the same PATTERN entry: `umbrella-daemon` adds `PATTERN-shed-stop-chain`, and this run only cites existing entries.
- Parent commits: `lyx loom peers` lists none since the fork.
