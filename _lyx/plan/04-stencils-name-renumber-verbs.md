# Card 4 — stencils-name-renumber-verbs

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils#wordingClaims`

**Intent:**
Tell the agents that edit a plan that the Card 3 verbs are the only way to insert or remove a card file (decision record D4).
Each stencil gains one sentence that names the verbs and points at their help, restating neither their bounds nor their envelope (`PATTERN-producer-pointer-rule`), one sentence per line:

- `loom-template-plan.md`, in `### On-disk layout` after "cards run `1..M` with no gaps.": "Insert or remove a card only with `lyx loom insert-card` or `lyx loom remove-card`, which renumber the later cards and whose `--help` says what they do, never by renaming, rewriting under a new name or deleting a card file by hand."
- `loom-template-rework.md`, as a new last bullet of `## Step 3 — Write a complete new plan`: "Insert or remove a card only with `lyx loom insert-card` or `lyx loom remove-card`, whose `--help` says what they do, never by renaming, rewriting under a new name or deleting a card file by hand."
- `loom-rubric-plan-review.md`, at the end of `## Write surface`, after the follow-up-card sentence: "A fix inserts or removes a card only with `lyx loom insert-card` or `lyx loom remove-card`, whose `--help` says what they do, never by renaming, rewriting under a new name or deleting a card file by hand."
  The rubric is the Plan-Burler fixer's channel, since its round prompt carries the rubric; the burler stencils stay unchanged.

`wordingClaims` gains, in the row of each of the three stencils, one `must` claim for "only with `lyx loom insert-card` or `lyx loom remove-card`" and one for "never by renaming, rewriting under a new name or deleting a card file by hand", the rubric's scoped with `section: "## Write surface"`, so `TestStencilClaims` fails if a later edit drops either.
No phrase names a command the agents' settings deny, so `PATTERN-no-denied-recovery`'s scan stays green.

**ImpactSummary:** three stencils gain one sentence each and the claims table six rows; no stencil marker, Go contract or other test changes.

**Commit:** 4: stencils: name insert-card and remove-card as the card insertion path
