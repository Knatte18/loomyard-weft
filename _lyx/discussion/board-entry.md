# Board entry `plan-refs-auto-rewrite`: Plan references follow renames and renumbers mechanically

Two places where a plan's references go stale and are fixed by hand or by a full review, though the change is mechanical.
Both are rewritten mechanically here.

## 1. A fork's glyph rename reaches the remaining cards (was note `plan-conflict-small-fixer`)

### Seen (gate-runs-once, 2026-10-24)

Batch 06's fork replaced `internal/gatecli#runGuardStep` with `guardStepRun`, as a deviation; its report warned that card 14 still named the old function.
`begin-batch 08` then refused with `plan_conflict: tree_moved` on card 14's glyph, and Webster's `on_reroute: Plan-Burler-Review` sent the run through a full Plan perch: Plan-Burler-Review, Plan-Burler-Fix and Plan-Bouncer, 11:45 to 12:01 local, before Webster resumed at batch 08.
The fix was one renamed glyph in one card, which the refusal named exactly.

### Seen in the code (orch, 2026-10-25)

`planglyph.DetectDrift` (`internal/planglyph/drift.go`) already repairs a rename after each batch: plan-wide `planparser.RewriteRefs`, a revalidating resolve, and one plan amendment per repair.
It acts only on quarry's exact tier, a rename quarry proves from the AST; a function renamed and changed at once is not proven, so nothing was repaired.
The fork knows what it renamed, but its report has no field for it: the digest's `deviations` is the scope guard's file list, and an older fork's `deviations` key is ignored.

### Decision (operator, 2026-10-25)

- A fork reports deviations as glyphs, not files: the batch report gets a field of renames, each an old glyph and its new glyph.
- Webster applies each reported rename to the cards not yet begun through the same repair path as `DetectDrift`'s exact tier: rewrite, revalidating resolve, amendment, re-baseline.
- Only a reported new glyph that does not resolve, or a missing glyph with no reported replacement, goes on to Plan-Burler-Review as today.
- No fixer agent, and no change to the loom recipe or its `on_reroute`.

## 2. insert-card and remove-card rewrite card references (was note `renumber-rewrites-card-refs`)

Split from task plan-check-lint-pattern-per-card (issue #866).

`lyx loom insert-card` and `remove-card` renumber the cards after the edit and report every prose line naming a moved card by its old number (`planparser.RenumberResult.ProseMentions`), leaving the fix to the writer.
A renumber spanning many cards then needs every reference fixed by hand, and repeating one doubles the cost.

Open question: a prose number may already mean the new numbering, so an automatic rewrite needs a form that marks a reference unambiguously, such as a card id `<N>-<slug>`, rather than rewriting bare numbers.
