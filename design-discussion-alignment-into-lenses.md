# Discussion scope check moves from the alignment agent into the perch lenses, with an escalating finding class

`discussion-alignment-into-lenses` · Note · enhancement, loom, burler, shed

## Seen

The alignment agent runs at two points of a loom run (`internal/alignment`): `parent-review`, after Discussion-Write's mechanical gate and before the Discussion perch, and `pr-gate`, after Publish.
At `parent-review` it checks the discussion against the board entry, the board, the in-flight runs' records (`lyx loom peers`) and the parent's landed commits, and approves, rejects or escalates to the orch.
The Discussion perch already covers most of that: lens `goal-scope` (in fans `discussion` and `twill-discussion`) checks the record against its board entry, and lens `board-fit` (in `twill-discussion`) checks it against every other board entry.
The name is stale too: the parent no longer reviews at that point, so neither the gate nor `lyx loom review` should say parent-review.

## Idea

Drop the alignment agent at the discussion point and keep it at `pr-gate`, where it approves the landing and names the board entries a run superseded:

- `board-fit` also reads the other in-flight runs' records through `lyx loom peers` and the parent's landed commits.
- A lens gets a finding class that escalates to the run's parent instead of going to the fixer, for a question only the operator can settle: a design conflict with another run, or doubt about what the task is for.
  The Bouncer routes such a finding up to the orch.
- The `parent-review` gate and the `lyx loom review` verbs go, and with them the parent-review name.

Gain: one cold agent spawn fewer per loom run.
Cost: the scope check no longer runs before the first perch round, so a round can be spent polishing a record whose scope then changes.

Discussed with the operator 2026-10-22; the operator agreed to record it as a note, the design is not yet decided.
