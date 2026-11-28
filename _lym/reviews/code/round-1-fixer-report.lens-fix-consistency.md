```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of this round's change set (F1–F4 edits and the landing summary).

**LOW**: `contracts/stencils/burler/burler-review-format.md`, the new bullet "A finding whose `fix` carries another reviewer's change it may not reject…"
- **Summary:** the bullet is an exception to the sibling bullet above it but does not say so.
- **Evidence:** the sibling reads "Never write a `fix` that rests on a fact the code disproves. Propose a fix that holds without that fact … when no such fix exists, raise the finding as class `decision`".
- The new bullet says such a finding "keeps that change and appends the fact the code disproves", so its `fix` does rest on a disproved fact. A reader has to infer that this overrides both the "Never" and the `decision` route.
- `burler-parent-consolidation.md` also fixes a kept parent finding's class to `design`, `scope` or `consistency`, so the `decision` route cannot apply to it either.
- Opening the bullet with "The one exception:" would settle it.

I found no other disagreement:
- The three first-caller sites are removed together.
- The summary matches the new step-fix and review-format wording.
- My search found no fixer-side rule about parent findings that conflicts with the new step-fix line.

Tool calls:
- One grep for parent-finding rules in `contracts/stencils` and `internal/burlerengine`, to look for a fixer-side twin of the new bullet. My context held only the consolidation stencil and the fixer step.
