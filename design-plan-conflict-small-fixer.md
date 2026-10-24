# Plan conflict from a moved tree: a small fixer before the Plan perch

`plan-conflict-small-fixer` · Note · enhancement, webster, loom

## Seen (gate-runs-once, 2026-10-24)

Batch 06's fork replaced `internal/gatecli#runGuardStep` with `guardStepRun`, as a deviation; its report warned that card 14 still named the old function.
`begin-batch 08` then refused with `plan_conflict: tree_moved` on card 14's glyph, and Webster's `on_reroute: Plan-Burler-Review` sent the run through a full Plan perch: Plan-Burler-Review, Plan-Burler-Fix and Plan-Bouncer, 11:45 to 12:01 local, before Webster resumed at batch 08.
The fix was one renamed glyph in one card, which the refusal named exactly.

## Idea (operator, 2026-10-24)

A `tree_moved` conflict whose refusal names the missing glyphs and cards goes to a small fixer agent first, not to a full review perch: it edits the named cards to match the tree, runs `lyx webster rebaseline --card NN`, and Webster begins the batch again.
Only a conflict the fixer cannot settle, or one that changes a card's intent, goes on to Plan-Burler-Review.
