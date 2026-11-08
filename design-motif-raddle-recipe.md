# One recipe that runs Motif and raddle in a row, with a perch on the split

`motif-raddle-recipe` · Note · feature, shed, loom, burler, arbiter, board

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
- Open: whether a Motif that stops at the Sketch stays as a recipe of its own, and what the combined recipe is called.
- Size: shed, raddle, Burler and arbiter, so a Motif of its own.
