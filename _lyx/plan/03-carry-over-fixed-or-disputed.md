# Card 3 — carry-over-fixed-or-disputed

**Edit:**
- `internal/discussionparser#renderCarryOver`
- `internal/discussionparser#CarryOverConverged`
- `internal/discussionparser#TestWriteCarryOver_RendersConvergedAndAcceptedEntries`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-rework.md`

**Intent:**
Change the converged closing text `renderCarryOver` renders from `converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix` to `converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which`.
The accepted-circling closing text, the entry's structure, the marker lines and the paths line are unchanged.
The `CarryOverConverged` doc comment says the same: each listed finding was fixed or disputed by the round's fixer, no fresh reviewer has seen the result, and the fixer report says which.

In `TestWriteCarryOver_RendersConvergedAndAcceptedEntries`, the existing `converged` row's expected bullet becomes the new closing text; the `accepted` row is unchanged, and no test is added.
This card greps for the old text: the only assertion of it is that converged row.

In `loom-template-plan.md` (Step 1) and `loom-template-rework.md` (step 3 of the rework sequence), the carry-over sentence says an entry between the marker lines lists review findings the closing round's fixer fixed or disputed, unseen by a fresh reviewer, and that the entry's fixer report says which; the following sentence, that the reader checks each one against the code and never plans an entry as scope, stays so the pinned claim `never plan an entry as scope` holds.
No marker is added or removed in either stencil.
Every edited comment and markdown line uses semantic line breaks.

**ImpactSummary:** One rendered string and one const comment in discussionparser, the one test row that pins the string, and one sentence in each of two loom stencils; no caller signature changes.

**Commit:** 3: Converged carry-over entry says fixed or disputed, fixer report says which
