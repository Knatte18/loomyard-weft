# Discussion scope check into the perch lenses, outdated entries to the parent, Merge-In renamed Parent-Align

`discussion-scope-in-lenses` · Task · enhancement, loom, burler, shed, stencil · running · Merge-In

Three notes bundled into one task (operator, 2026-10-25): the Discussion scope check moves into the perch lenses (was `discussion-alignment-into-lenses`), a plainly outdated entry goes to the parent (was `outdated-entry-to-parent`), and `Merge-In` is renamed `Parent-Align` (was `merge-in-parent-align`).

## Part 1: the Discussion scope check moves into the perch lenses

### Seen

The alignment agent runs at two points of a loom run (`internal/alignment`): `parent-review`, after Discussion-Write's mechanical gate and before the Discussion perch, and `pr-gate`, after Publish.
At `parent-review` it checks the discussion against the board entry, the board, the in-flight runs' records (`lyx loom peers`) and the parent's landed commits, and approves, rejects or escalates to the orch.
The Discussion perch already covers most of that: lens `goal-scope` (in fans `discussion` and `twill-discussion`) checks the record against its board entry, and lens `board-fit` (in `twill-discussion`) checks it against every other board entry.
The name is stale too: the parent no longer reviews at that point, so neither the gate nor `lyx loom review` should say parent-review.

### Design

Operator decision (2026-10-25): drop the alignment agent at the discussion point, since the Discussion perch does the same work as part of a lens; keep it at `pr-gate`, where it approves the landing.

- `board-fit` also reads the other in-flight runs' records through `lyx loom peers` and the parent's landed commits.
- A lens gets a finding class that escalates to the run's parent instead of going to the fixer, for a question only the operator can settle: a design conflict with another run, or doubt about what the task is for.
  The Bouncer routes such a finding up to the orch.
  Since `decision-reaches-agents` (2e265a4fc) the `decision` finding class exists and the Burler fix row posts it to the parent and keeps it from the fixer; what is left is the Bouncer's routing.
- The `parent-review` gate and the `lyx loom review` verbs go, and with them the parent-review name.

Gain: one cold agent spawn fewer per loom run.
Cost: the scope check no longer runs before the first perch round, so a round can be spent polishing a record whose scope then changes.

## Part 2: a plainly outdated entry goes to the parent

### Seen

A task's board entry is written when it is promoted, and can sit for days while other work lands and other runs start.
Its text reaches Discussion-Write and Darn-Write as it stands, so a scope that overlaps, contradicts or has been overtaken by other planned or landed work is found late, after a whole Discussion or a whole darn change has been written from it.

### Design (operator, 2026-10-24)

No new row runs before Discussion, and the board's run lock gets no exemption: an outdated entry is caught by work the run already does, and goes to the parent.

- Loom: the Discussion review's `board-fit` lens gets a finding class "the entry is plainly outdated" (overtaken by landed work, contradicted by the board or another run).
  Such a finding escalates to the parent orch instead of going to the fixer, through the escalating finding class of part 1.
- Darn: it has no Discussion review, and Darn-Write reads the entry and writes code in one go.
  So Darn-Write itself checks the entry text against the parent's recent landed commits and the board before it starts, and when the text is plainly outdated it asks the parent and waits instead of guessing.

Only a plainly outdated entry goes to the parent; an ordinary small misalignment is settled by the writer as today.

## Part 3: rename `Merge-In` to `Parent-Align`

### Seen (2026-10-24)

Since `loom-tail-review` landed, loom's `Merge-In` row does more than merge: it takes the parent in, and when the parent brought commits in, or the merge conflicts or fails its gate, its fixer resolves the conflicts, makes the tests green and checks that the task's work still matches its decision record against what came in, escalating a decision that collides with landed work.
Without new parent commits and with a green gate it runs no fixer.
`Finalize` keeps its own small merge-in under the parent lock, as a last sync.

### Design (operator, 2026-10-24)

Rename the row `Merge-In` to `Parent-Align`, so its name says it aligns the task with its parent, not only merges.
The rename covers the recipe row in loom and darn, the producer and engine names, the `merge_*` config keys it reads, the stencils and docs.
Row names are durable resume identities, so a run parked on `Merge-In` is restarted, as the recipe already says for retired rows.
