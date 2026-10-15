# Darn: a decision record for mid-run scope corrections, and PR-Align

`twb-d0-sf` · Note · bug · paused · Discussion-Bouncer

## Seen (2026-10-13, `webster-pack-last`)

The task was scoped with the wrong variant (the pack at the end of the fork prompt) instead of the operator's decision in `glyph-use-audit` (the pack in its own file, the prompt ending with one line to read it).
The operator corrected the scope mid-run, and the orch confirmed it to Darn-Write, which built the own-file design (`645d6093b`).
The Darn review is a separate session that reads the scope from the board body, which still held the old variant: the entry was claimed, so the body could not be edited.
Its round-1 finding F1 (BLOCKING, design) called the own-file design out of scope, and the fix commit `f1ce95ce5` reverted it.
PR-Gate caught it and the PR was rejected, at the cost of a review round, a fix round and a rework round.

## The weakness (operator, 2026-10-13)

It is not good that a task's description cannot be changed while it runs.
A scope correction mid-run has no place every later session reads:
- a claimed board entry refuses any scope edit;
- a darn run has no decision record (`lyx loom decision add` is refused);
- a message to a session reaches only that session.

## Loom has the way, darn does not (operator, 2026-10-13)

In a loom run the parent records a correction with `lyx loom decision add --by parent` once Discussion-Write has written the decision record, and the later rows read that record.
Darn skips Discussion and has no decision record, so the gap is darn's.
Loom keeps a narrower gap: before Discussion-Write has written the record, `decision add` is refused.

## Direction (operator, 2026-10-13)

Darn stays one combined agent; turning it into a loom without perches (Discussion-Write, Plan-Write and Webster in turn) was considered and dropped, since each row would explore the code anew and the run would be slower and dearer.
Instead Darn-Write writes a short decision record in the same session before it writes code: the scope as it reads it and the calls it makes, in loom's record format.
Then `lyx loom decision add --by parent` works on a darn run once that record exists, and the Darn review reads the record and holds it authoritative over the board body.
First check where the Darn review reads its scope from today (the board body or a copy), which sets how much the change touches.

## Darn also lacks PR-Align (operator, 2026-10-13)

The loom recipe's PR-Align row spawns the alignment agent, which approves and lands the pull request itself (`alignment_pr_approve`); `contracts/recipes/darn-recipe.yaml` has no such row, so every darn run stops at PR-Gate for the orch.
The operator wants PR-Align in darn too.
Check first whether darn left it out on purpose, and what the alignment agent reads that darn lacks (the decision record above may be one).

## Other options considered

- Let the parent (orch or operator) edit a claimed entry's body, or append a scope amendment to it, recorded with who and when, and make every row that reads the scope read the current body.
- Or give darn the decision record loom has, so `lyx loom decision add --by parent` works in every recipe, and have every reviewer read it.
- Either way a reviewer must treat the amended scope as authoritative over the original.

