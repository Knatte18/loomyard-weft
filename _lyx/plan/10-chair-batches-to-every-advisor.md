# Card 10 — chair-batches-to-every-advisor

**Edit:**
- `contracts/stencils/loom/loom-template-discussion-chair.md`
- `contracts/stencils#wordingClaims`

**Uses:**
- `contracts/stencils/loom/loom-template-discussion.md`

**Intent:** Replace the area assignment in the chair stencil's `## Working with your advisors` section with the operator's protocol: every question batch goes to every advisor, and in autonomous mode the chair settles each question only after weighing every advisor's answer to it.
The first bullet of the section, which today sends each advisor its first batch after Step 2 and, with more than one advisor, assigns each an area of the design to study, becomes: after Step 2, form your question batches exactly as Step 3 says, each question with your recommended answer and its alternatives, and send every batch to every advisor; no advisor owns an area, since each may find a different weakness.
The second bullet, which today sends every interview batch to the advisors before the chair settles it in autonomous mode, becomes: in autonomous mode, settle each question only after weighing every advisor's answer to it, and a pick made with no usable answer stays an auto-pick.
The interactive-mode bullet, the `## Interview` and question-ledger bullets, the notes-file bullet and the Step 6 bullet stay word for word, and the header comment stays, since the stencil still declares no marker of its own.
No sentence of the section uses an ask-the-operator phrase without a prohibition word, since the parent-directive scan reads every registered stencil.
Grep the repository for `an area of the design` and `assign each an area`: the only hit today is the stencil line itself, so no test asserts the old text, and the greps are the card's proof of that.
In `wordingClaims`, the chair-stencil row gains three claims under its `## Working with your advisors` section: a `must` on `send every batch to every advisor`, whose why is that every advisor sees every question since each may find a different weakness; a `must` on `after weighing every advisor's answer`, whose why is that the chair settles a question on all advisors' answers, not one advisor's area; and a `mustNot` on `an area of the design`, whose why is that the retired area assignment never returns.
The two `must` strings are spelled in the stencil exactly as the claims quote them.
`TestStencils_TopLevelMarkers`, the parent-directive and edit-directive scans and the registry on-disk test cover the edited stencil without change, and the package's top-level test count is unchanged, so `cmd/lyx/testdata/test-budget.yaml` is not touched.

**ImpactSummary:** One embedded stencil's advisors section and one claims row; the included discussion stencil, the single-agent prompt, the seat directives, the Go-held `mode_rules` text and every other stencil are untouched, so only the chair's rendered prompt changes.

**Commit:** 10: chair stencil sends every question batch to every advisor
