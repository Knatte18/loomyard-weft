# Arras: one recipe that runs Motif and raddle in a row, with a perch on the split

`motif-raddle-recipe` · Task · feature, shed, loom, burler, arbiter, board · running · Worktree-Create

## Seen (operator and orch, 2026-11-08)

Going from a note to tasks through Motif and raddle takes three manual steps today: the operator promotes the Sketch to raddle, approves the raddle proposal in the run pane, and releases each filed task.
Most other steps in lym run on their own, with dedicated checkers instead of the orch: the Burler perches review in rounds and the arbiter settles stops.
The raddle approval was manual only while raddle was being tuned (operator, 2026-11-08).
On 2026-11-08 the first raddle proposal for `slack-v2` missed the operator's changes made after the Discussion, because they sat in the entry body and not in the decision record; review caught it, the checks did not.

## Idea (operator, 2026-11-08)

- One recipe runs Motif and raddle in a row: Discussion with its perch, then the raddle split, then a perch on the split, then the filed tasks as drafts.
- The split perch reviews the proposal against the Sketch, its decision record and every decision recorded after the Discussion; raddle fixes what it finds; the arbiter settles stops, and only what it cannot settle reaches the orch.
- Operator decisions made after the Discussion go into the record through `lym loom decision add --by operator`, so the split and its perch see them.
- The operator only releases the filed tasks.
- The recipe is called `arras` (operator, 2026-11-08). `cartoon` stays reserved for a later Plan block, since a cartoon is an artifact, the drawing the weaver follows, not the way to it; the weaving glossary (`docs/reference/weaving-glossary.md`) lists `arras` as free and is updated with it.
- `arras` is the default for a tapestry-sized note; Motif alone stays for a design decided now but split later, on code still changing, such as `slack-machine-bridge` waiting for `slack-push-channels`.
- The shipped rule that a tapestry-sized note runs as a Motif, then raddle (in the `ly:board-policy` skill and the orch text) is rewritten to name `arras`.
- Size: shed, raddle, Burler and arbiter, so a Motif of its own.

## When an arras run ends (operator, 2026-11-08)

- One task: the split files a single task, drafted, carrying the decision record; the arras entry itself is done and pruned.
- A family of tasks: the split files them into a new project named after the arras entry's slug, so the board shows they are the result of that arras run; each task carries its `depends_on` and its part of the decision record, and the project lists their order.
- Each task of a family is an ordinary darn or tapestry with its own perch and its own landing on `main`; a darn is a task of fair size that warrants its own review, cheaper and more local than a tapestry, and tested on its own before the next starts (operator, 2026-11-08).
  A closed project whose darns skip their perch, with one perch over the whole project at the end, may be tried later; it is not part of this design.
- For the Discussion to settle with the operator: whether releasing the project releases all its tasks at once, in their `depends_on` order, and whether the project keeps the whole decision record for later reading. The orch proposed yes to both.
